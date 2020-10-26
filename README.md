# Multi-threading
**Multi-threading** is a process of executing multiple threads simultaneously.

**Process**: is an instance of a program. / **Thread**: is a lightweight sub-process.

    public class Main {
        public static void main(String[] args) {
            // 2 threads (1. main thread, 2. garbage collector thread)
            System.out.println("Used Threads: " + Thread.activeCount());
            // depend on your cpu (4 or 8 or more)
            System.out.println("Available Threads: " + Runtime.getRuntime().availableProcessors());
        }
    }
    
### Start Thread
    
    public class Main {
        public static void main(String[] args) {
            System.out.println("Current Thread: " + Thread.currentThread().getName()); // main thread

            Thread thread = new Thread(new FileDownloader());
            thread.start();
        }
    }

    public class FileDownloader implements Runnable {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + ": File Downloading ..."); // Thread-0
        }
    }    

