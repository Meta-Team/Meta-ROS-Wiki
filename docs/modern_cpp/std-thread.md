# `std::thread` and `std::jthread`

`std::thread` was added to C++ in C++ 11, so it's kind of "Modern" C++. But in reality, it's a mere wrapper around `pthread` functions and faces limitations that make it less ideal for concurrent programming. The most notable limitation is that `std::thread` is not a RAII object. Basically it means that the actual thread object is not terminated when the `std::thread` object is destroyed. 

## A simple example (that doesn't shutdown correctly)

It's common to write a simple worker (e.g. motor driver) class that starts a thread using hammers (e.g. CAN/UART driver). Here's how you might write it:

```cpp
class Worker {
public:
    Worker() {
        thread_ = std::thread(&Worker::run, this);
    }
    ~Worker() = default;

private:
    void run() {
        while (true) {
            hammer_.hit();
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
    }

    std::thread thread_;
    Hammer hammer_;
};
```

The problem with this code is that when the `Worker` object is destroyed, the actual thread is still running. But the `Hammer` object has been destroyed, you can easily get a segmentation fault. In practice, this is where most of your SIGSEGVs during shutdown come from. You may ignore it, but it's going to be a big problem when you're working with multiple `Worker`s as the other workers may not be able to trigger their destructors and shutdown some actual hardware (especially dangerous if you cannot properly shutdown motors).

## Shutdown threads correctly with `std::thread`

To shutdown threads correctly, you need to have a way to signal the thread to stop. The most common way to do this is to use a `std::atomic<bool>` flag. Here's how you might write it:

```cpp
class Worker {
public:
    Worker() {
        thread_ = std::thread(&Worker::run, this);
    }
    ~Worker() {
        stop_ = true;
        thread_.join();
    }

private:
    void run() {
        while (!stop_) {
            hammer_.hit();
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
    }

    std::thread thread_;
    Hammer hammer_;
    std::atomic<bool> stop_{false};
};
```

This code solves the problem of the previous example. When the `Worker` object is destroyed, the `stop_` flag is set to `true`, and the thread will exit the while loop in the next iteration. The `join()` function blocks until the thread is finished. This way, you can ensure that the thread is properly terminated before the `Worker` object is destroyed.

## `std::jthread` in C++20

The name of `std::jthread` stands for "joinable thread". It's a new addition to C++20 that automatically joins when the object is destroyed. Here's how you might write the previous example with `std::jthread`:

```cpp
class Worker {
public:
    Worker() {
        thread_ = std::jthread(&Worker::run, this);
    }
    ~Worker() = default;
    
private:
    void run() {
        while (true) {
            hammer_.hit();
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
    }

    std::jthread thread_;
    Hammer hammer_;
};
```

But this code is actually incorrect. Because the `hammer_` object is destroyed before the `thread_` object (C++ destruction happens from bottom to top), you can still get a segmentation fault. You can of course easily fix this by swapping the order of the member variables:

```cpp
class Worker {
public:
    Worker() {
        thread_ = std::jthread(&Worker::run, this);
    }
    ~Worker() = default;
    
private:
    void run() {
        while (true) {
            hammer_.hit();
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
    }

    Hammer hammer_;
    std::jthread thread_;
};
```
## `std::stop_token`

Another addition of `std::jthread` is the `std::stop_token`. It's a standardized way to stop a thread. Here's how you might write the previous example with `std::stop_token`:

```cpp
class Worker {
public:
    Worker() {
        thread_ = std::jthread([this](std::stop_token s) {run(s);});
    }
    ~Worker() = default;

private:
    void run(std::stop_token st) {
        while (!st.stop_requested()) {
            hammer_.hit();
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
    }

    Hammer hammer_;
    std::jthread thread_;
};
```
Note that to use `std::stop_token` with `std::jthread`, you need to create a function that (only) takes  a `std::stop_token` as an argument because `std::jthread` must pass a `std::stop_token` to the function. But unfortunately, all the member functions of a class takes an additional `this` pointer as an argument, so a workaround is to use a lambda function that captures `this` and takes a `std::stop_token` as an argument.
