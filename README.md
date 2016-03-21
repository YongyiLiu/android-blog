# android-blog(内存泄露问题)【转载】
Android 编程所使用的 Java 是一门使用垃圾收集器（GC, garbage collection）来自动管理内存的语言，它使得我们不再需要手动调用代码来进行内存
回收。那么它是如何判断的呢？简单说，如果一个对象，从它的根节点开始不可达的话，那么这个对象就是没有引用的了，是会被垃圾收集器回收的，其中，所谓的“根节点” 往往是一个线程，比如主线程。因此，如果一个对象从它的根节点开始是可达的有引用的，但实际上它已经没有再使用了，是无用的，这样的对象就是内存泄漏的对象，它会在内存中占据我们应用程序原本就不是很多的内存，导致程序变慢，甚至内存溢出（OOM）程序崩溃。

内存泄漏的原因并不难理解，但仅管知道它的存在，往往我们还是会不知觉中写出致使内存泄漏的代码。在 Android编程中，也是有许多情景容易导致内
存泄漏，以下将一一列举一些我所知道的内存泄漏案例，从这些例子中应该能更加直观了解怎么导致了内存泄漏，从而在编程过程中去避免。

#静态变量造成内存泄漏【准确地说是静态变量引用】

首先，比较简单的一种情况是，静态变量致使内存泄漏，说到静态变量，我们至少得了解其生命周期才能彻底明白。静态变量的生命周期，起始于类的加
载，终止于类的释放。对于 Android 而言，程序也是从一个main方法进入，开始了主线程的工作，如果一个类在主线程或旁枝中被使用到，它就会被加载，反过来说，假如一个类存在于我们的项目中，但它从未被我们使用过，算是个孤岛，这时它是没有被加载的。一旦被加载，只有等到我们的 Android 应用进程结束它才会被卸载。
于是，当我们在 Activity 中声明一个静态变量引用了 Activity 自身，就会造成内存泄漏：
public class LeakActivity extends AppCompatActivity {
    private static Context sContext;
    @Override 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_leak);
        sContext = this;
    }
}
这样的代码会导致当这个 Activity 结束的时候，sContext 仍然持有它的引用，致使 Activity 无法回收。

**【解决方案】** 这个 Activity 的 onDestroy 时将 sContext 的值置空，或者避免使用静态变量这样的写法。
同样的，如果一个 Activity 的静态 field 变量内部获得了当前 Activity 的引用，比如我们经常会把 this 传给 View 之类的对象，这个对象若是静态的，并且没有在 Activity 生命周期结束之前置空的话，也会导致同样的问题。
