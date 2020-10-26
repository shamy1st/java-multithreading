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

### Pause Thread
    try {
        Thread.sleep(5000); // 5000 msec
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    
### Join Thread
Example: main thread waits "Thread-0" to finish.

    public class Main {
        public static void main(String[] args) {
            System.out.println("Current Thread: " + Thread.currentThread().getName());
            
            Thread thread = new Thread(new FileDownloader());
            thread.start();

            // main thread waits "Thread-0" to finish.
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("Thread main finished.");
        }
    }

    public class FileDownloader implements Runnable {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + ": File Downloading ...");

            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

### Interrupt Thread
Interrupt a thread doesn't breaks out it, but it raise a flag of interrupt and it is optional to thread to cancel or not.

    public class Main {
        public static void main(String[] args) {
            System.out.println("Current Thread: " + Thread.currentThread().getName());

            Thread thread = new Thread(new FileDownloader());
            thread.start();

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            thread.interrupt();

            System.out.println("Thread main finished.");
        }
    }

    public class FileDownloader implements Runnable {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + ": File Downloading ...");

            for(int i=0; i<Integer.MAX_VALUE; i++) {
                if(Thread.currentThread().isInterrupted())
                    return;

                System.out.println("Download byte " + i);
            }
        }
    }

### Concurrency Problems
1. **Race Condition** multiple threads try to change shared data at the same time.
2. **Visibility Problem** one thread changes a shared data but the change does not visibile to other threads.

### Race Condition
multiple threads try to change shared data at the same time.

    public class Main {
        public static void main(String[] args) {
            DownloadStatus status = new DownloadStatus();

            List<Thread> threads = new ArrayList<>();
            for(int i=0; i<10; i++) {
                Thread thread = new Thread(new FileDownloader(status));
                thread.start();
                threads.add(thread);
            }

            threads.forEach(thread -> {
                try {
                    thread.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });

            // expected: 10000, but it always print less (ex: 81395) because of race condition problem
            System.out.println("Total Bytes: " + status.getTotalBytes());
        }
    }

    public class FileDownloader implements Runnable {
        private DownloadStatus status;

        public FileDownloader(DownloadStatus status) {
            this.status = status;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + ": File Downloading ...");

            for(int i=0; i<10_000; i++) {
                if(Thread.currentThread().isInterrupted())
                    return;

                status.incrementTotalBytes();
            }

            System.out.println(Thread.currentThread().getName() + ": Download complete!");
        }
    }

    public class DownloadStatus {
        private int totalBytes;

        public int getTotalBytes() {
            return totalBytes;
        }

        public void incrementTotalBytes() {
            this.totalBytes++;
        }
    }
    
### Race Condition - Thread Safety Strategies

1. **Confinement** no shared data, each thread have its own data.
2. **Immutability** unmodified objects.
3. **Synchronization** prevent multiple threads to access the same object concurrently.
    * use locks and leads to deadlock which crash the program, should be avoided.
4. **Atomic Objects** write an object as a single atomic operation. (example: AtomicInteger)
5. **Partitioning** into segments that can be accessed concurrently. 

### Confinement
No shared data, each thread have its own data.

In the previous example each thread will have its **DownloadStatus** object and after all threads finish will sum all **totalBytes** together.

### Synchronization
Prevent multiple threads to access the same object concurrently.

* **Lock**

        public class DownloadStatus {
            private int totalBytes;
            private Lock lock = new ReentrantLock();

            public int getTotalBytes() {
                return totalBytes;
            }

            public void incrementTotalBytes() {
                lock.lock();
                try {
                    this.totalBytes++;
                } finally {
                    // to avoid not doing unlock() when crash, which cause deadlock.
                    lock.unlock();
                }
            }
        }
    
* **Synchronized Keyword** (bad practice)

        public synchronized void incrementTotalBytes() {
            this.totalBytes++;
        }

        //same as 

        public void incrementTotalBytes() {
            synchronized(this) {
                this.totalBytes++;
            }
        }

    Try to avoid prevoius two ways because of passing **this** to synchronized block cause locking on the current object and if you have two synchronized blocks one of them can't access its block while the another block accessing its block!

    The right way to use synchronized:

        public class DownloadStatus {
            private int totalBytes;
            private Object totalBytesLock = new Object();

            public int getTotalBytes() {
                return totalBytes;
            }

            public void incrementTotalBytes() {
                synchronized(totalBytesLock) {
                    this.totalBytes++;
                }
            }
        }

### Atomic Objects
    public class DownloadStatus {
        private AtomicInteger totalBytes = new AtomicInteger();

        public int getTotalBytes() {
            return totalBytes.get();
        }

        public void incrementTotalBytes() {
            this.totalBytes.incrementAndGet();
        }
    }

### Visibility Problem - Thread Safety Strategies
1. **Volatile Keyword**
