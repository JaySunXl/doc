




一个Thread中只有一个ThreadLocalMap,
一个ThreadLocalMap中可以有多个ThreadLocal对象,
其中一个ThreadLocal对象对应一个ThreadLocalMap中的一个Entry
(也就是说:一个Thread可以依附有多个ThreadLocal对象)


ThreadLocal在没有线程池使用的情况下,正常情况下不会存在内存泄露,但是如果使用了线程池的话,就依赖于线程池的实现,如果线程池不销毁线程的话,那么就会存在内存泄露