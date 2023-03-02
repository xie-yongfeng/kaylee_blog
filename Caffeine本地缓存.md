---


---

<h2 id="一、简介"><span class="prefix"></span><span class="content">一、简介</span><span class="suffix"></span></h2>
<p>基于Java8开发，提供了近乎最佳的命中率的高性能本地缓存组件</p>
<p>不管是并发读、并发写还是并发读写的场景下，Caffeine的性能都优于其他缓存组件</p>
<h2 id="二、原理"><span class="prefix"></span><span class="content">二、原理</span><span class="suffix"></span></h2>
<h3 id="、淘汰算法"><span class="prefix"></span><span class="content">2.1、淘汰算法</span><span class="suffix"></span></h3>
<p>对于Java进程缓存可以通过Hashmap来实现，但内存是有限的，有合适的算法辅助我们淘汰掉使用价值相对不高的数据</p>
<p>Caffeine的缓存清除动作是<strong>触发式</strong>的，它可能发生在读、写请求后。这个动作在Caffeine中是<strong>异步执行</strong>的，默认执行的线程池是ForkJoinPool.commonPool()。</p>
<blockquote>
<p>常见的淘汰算法有</p>
</blockquote>
<ol>
<li>FIFO(First In First Out)：先进先出</li>
<li>LRU(Least Recently Used)：最近最久未使用</li>
<li>LFU(Least Frequently Used)：最近最少频率使用</li>
</ol>
<h4 id="fifofirst-in-first-out：先进先出"><span class="prefix"></span><span class="content">FIFO(First In First Out)：先进先出</span><span class="suffix"></span></h4>
<p>旨在先进先出的思想，当缓存满时，会淘汰先进入缓存的数据，该算法较简单</p>
<h4 id="lruleast-recently-used：最近最久未使用"><span class="prefix"></span><span class="content">LRU(Least Recently Used)：最近最久未使用</span><span class="suffix"></span></h4>
<p>旨在最近访问过的数据在将来可能被访问的几率会更高，通常使用链表来实现。</p>
<p>如果数据被访问或添加时，会将数据移动到链表的头部，<strong>链表头部数据为热数据，反之</strong>，当数据满时淘汰尾部数据。</p>
<p><img src="https://upload-images.jianshu.io/upload_images/11345047-7a5b2173d8f94152.png?imageMogr2/auto-orient/strip%7CimageView2/2/format/webp" alt=""></p>
<h4 id="lfuleast-frequently-used：最近最少频率使用"><span class="prefix"></span><span class="content">LFU(Least Frequently Used)：最近最少频率使用</span><span class="suffix"></span></h4>
<p>旨在使用频率高的数据将来可能被使用的几率更高，当该算法需要去维护一套统计每个元素的使用次数，会浪费内存空间。</p>
<p><img src="https://upload-images.jianshu.io/upload_images/11345047-e3cd6631f61abf30.png?imageMogr2/auto-orient/strip%7CimageView2/2/format/webp" alt=""></p>
<h4 id="tinylfu算法"><span class="prefix"></span><span class="content">TinyLFU算法</span><span class="suffix"></span></h4>
<p>Caffeine采用了一种结合LRU和LUF优点的算法：TinyLFU</p>
<blockquote>
<p>优点</p>
</blockquote>
<p>高命中率、低内存使用</p>
<p>采用 Count–Min Sketch 算法降低频率信息带来的内存消耗；</p>
<p>维护一个PK机制保障新上的热点数据能够缓存。</p>
<blockquote>
<p>统计元素的使用次数</p>
</blockquote>
<p>Count–Min Sketch算法类似布隆过滤器（Bloom filter）思想</p>
<p>存储数据时，对key进行4次hash算法后，存储到不同 long 型数字的16份中某个位置</p>
<p>当去每个key的访问次数时，会比较所有位置上的频率值，取最小值。对于达到最大用户频率的key，会对所有的频率值进行reset，除以2。</p>
<blockquote>
<p>元素的使用次数存储方式</p>
</blockquote>
<p>如下图缓存访问频率存储主要分为两大部分，即 LRU 和 Segmented LRU 。</p>
<p>新访问的数据会进入第一个 LRU，在 Caffeine 里叫 WindowDeque。</p>
<p>当 WindowDeque 满时，会进入 Segmented LRU 中的 ProbationDeque，</p>
<p>在后续被访问到时，它会被提升到 ProtectedDeque。</p>
<p>当 ProtectedDeque 满时，会有数据降级到 ProbationDeque 。</p>
<p><img src="https://pic2.zhimg.com/v2-28d17f352994e9e3dcdfe10fb3669a5d_r.jpg" alt=""></p>
<blockquote>
<p>淘汰流程</p>
</blockquote>
<p>数据需要淘汰的时候，对 ProbationDeque 中的数据进行淘汰。</p>
<p>具体淘汰机制：取ProbationDeque 中的队首和队尾进行 PK，队首数据是最先进入队列的，称为受害者，队尾的数据称为攻击者，比较两者 频率大小，大胜小汰。</p>
<h3 id="、读写优化"><span class="prefix"></span><span class="content">2.2、读写优化</span><span class="suffix"></span></h3>
<p>Caffeine认为<strong>读操作是频繁的，而写操作时偶尔</strong>，读写操作都是使用<strong>异步线程去更新频率信息</strong></p>
<p>Caffeine的底层数据存储采用<strong>ConcurrentHashMap</strong>。</p>
<p>针对读写请求后的异步操作（清除缓存、调整LRU队列顺序等操作），Caffeine分别使用了<strong>ReadBuffer</strong>和<strong>WriterBuffer</strong>。使用Buffer一方面能够合并操作，另一方面可以避免锁争用的问题。</p>
<h4 id="读缓冲（readbuffer"><span class="prefix"></span><span class="content">读缓冲（ReadBuffer)</span><span class="suffix"></span></h4>
<p>传统的缓存实现将会为每个操作加锁，以便能够安全的对每个访问队列的元素进行排序。一种优化方案是将每个操作按序加入到缓冲区中进行批处理操作。读完把数据放到环形队列 RingBuffer 中，为了减少读并发，采用多个 RingBuffer，每个线程都有对应的 RingBuffer。环形队列是一个定长数组，提供高性能的能力并最大程度上减少了 GC所带来的性能开销。数据丢到队列之后就返回读取结果，类似于数据库的WAL机制，和ConcurrentHashMap 读取数据相比，仅仅多了把数据放到队列这一步。异步线程并发读取 RingBuffer 数组，更新访问信息。</p>
<h4 id="写缓冲（writerbuffer"><span class="prefix"></span><span class="content">写缓冲（WriterBuffer)</span><span class="suffix"></span></h4>
<p>与读缓冲类似，写缓冲是为了储存写事件。读缓冲中的事件主要是为了优化驱逐策略的命中率，因此读缓冲中的事件完整程度允许一定程度的有损。但是写缓冲并不允许数据的丢失，因此其必须实现为一个安全的队列。Caffeine 写是把数据放入MpscGrowableArrayQueue 阻塞队列中，它参考了JCTools里的MpscGrowableArrayQueue ，是针对 MPSC- 多生产者单消费者（Multi-Producer &amp; Single-Consumer）场景的高性能实现。多个生产者同时并发地写入队列是线程安全的，但是同一时刻只允许一个消费者消费队列。</p>
<h2 id="三、使用"><span class="prefix"></span><span class="content">三、使用</span><span class="suffix"></span></h2>
<blockquote>
<p>使用场景</p>
</blockquote>
<p>配合Redis做一级缓存</p>
<blockquote>
<p>缓存的解决方案一般有三种</p>
</blockquote>
<ol>
<li>本地缓存：如Caffeine、Ebcache、适合单机系统，速度最快，但是容量有限，而且重启系统后缓存丢失</li>
<li>集中式缓存，如Redis、Memcached； 适合分布式系统，解决了容量、重启丢失缓存等问题，但是当访问量极大时，往往性能不是首要考虑的问题，而是带宽。现象就是 Redis 服务负载不高，但是由于机器网卡带宽跑满，导致数据读取非常慢</li>
<li>第三种方案就是结合以上2种方案的二级缓存应运而生，以内存缓存作为一级缓存、集中式缓存作为二级缓存</li>
</ol>
<blockquote>
<p>配置说明</p>
</blockquote>
<ul>
<li>initialCapacity=[integer]: 初始的缓存空间大小</li>
<li>maximumSize=[long]: 缓存的最大条数</li>
<li>maximumWeight=[long]: 缓存的最大权重</li>
<li>expireAfterAccess=[duration]: 最后一次写入或访问后经过固定时间过期</li>
<li>expireAfterWrite=[duration]: 最后一次写入后经过固定时间过期</li>
<li>refreshAfterWrite=[duration]: 创建缓存或者最近一次更新缓存后经过固定的时间间隔，刷新缓存</li>
<li>weakKeys: 打开key的弱引用</li>
<li>weakValues：打开value的弱引用</li>
<li>softValues：打开value的软引用</li>
<li>recordStats：开发统计功能</li>
</ul>
<blockquote>
<p>注意</p>
</blockquote>
<ul>
<li>expireAfterWrite和expireAfterAccess同事存在时，以expireAfterWrite为准。</li>
<li>maximumSize和maximumWeight不可以同时使用</li>
<li>weakValues和softValues不可以同时使用</li>
</ul>
<h3 id="、常规使用"><span class="prefix"></span><span class="content">3.1、常规使用</span><span class="suffix"></span></h3>
<h4 id="依赖"><span class="prefix"></span><span class="content">依赖</span><span class="suffix"></span></h4>
<pre class=" language-xml"><code class="prism  language-xml"><span class="token comment">&lt;!-- https://mvnrepository.com/artifact/com.github.ben-manes.caffeine/caffeine --&gt;</span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>dependency</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>groupId</span><span class="token punctuation">&gt;</span></span>com.github.ben-manes.caffeine<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>groupId</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>artifactId</span><span class="token punctuation">&gt;</span></span>caffeine<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>artifactId</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>version</span><span class="token punctuation">&gt;</span></span>3.1.1<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>version</span><span class="token punctuation">&gt;</span></span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>dependency</span><span class="token punctuation">&gt;</span></span>
</code></pre>
<h4 id="缓存填充策略"><span class="prefix"></span><span class="content">缓存填充策略</span><span class="suffix"></span></h4>
<p>Caffeine 的三种缓存填充策略：<strong>手动、同步加载和异步加载</strong></p>
<h5 id="手动加载"><span class="prefix"></span><span class="content">手动加载</span><span class="suffix"></span></h5>
<p>手动控制缓存的增删改处理，主动增加、获取以及依据函数式更新缓存；</p>
<pre class=" language-java"><code class="prism  language-java"><span class="token comment">/*
    手动填充测试
     */</span>
    <span class="token keyword">public</span> <span class="token keyword">void</span> <span class="token function">cacheTest</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        Cache<span class="token operator">&lt;</span>Object<span class="token punctuation">,</span> Object<span class="token operator">&gt;</span> cache <span class="token operator">=</span> Caffeine<span class="token punctuation">.</span><span class="token function">newBuilder</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">initialCapacity</span><span class="token punctuation">(</span><span class="token number">100</span><span class="token punctuation">)</span> <span class="token comment">// 初始缓存大小</span>
                <span class="token punctuation">.</span><span class="token function">maximumSize</span><span class="token punctuation">(</span><span class="token number">100</span><span class="token punctuation">)</span> <span class="token comment">// 最大缓存条数</span>
                <span class="token punctuation">.</span><span class="token function">expireAfterAccess</span><span class="token punctuation">(</span>100L<span class="token punctuation">,</span> TimeUnit<span class="token punctuation">.</span>SECONDS<span class="token punctuation">)</span> <span class="token comment">// 最后一次写入或更新固定时间过期，100秒过期</span>
                <span class="token punctuation">.</span><span class="token function">build</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token comment">// 存储数据</span>
        cache<span class="token punctuation">.</span><span class="token function">put</span><span class="token punctuation">(</span><span class="token string">"test"</span><span class="token punctuation">,</span><span class="token string">"test"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token comment">// 获取缓存。如果不存在，返回null</span>
        System<span class="token punctuation">.</span>out<span class="token punctuation">.</span><span class="token function">println</span><span class="token punctuation">(</span>cache<span class="token punctuation">.</span><span class="token function">getIfPresent</span><span class="token punctuation">(</span><span class="token string">"test"</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        
        <span class="token comment">// 清除缓存</span>
        cache<span class="token punctuation">.</span><span class="token function">invalidate</span><span class="token punctuation">(</span><span class="token string">"test"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token comment">// 清除全部缓存</span>
        cache<span class="token punctuation">.</span><span class="token function">invalidateAll</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
</code></pre>
<h5 id="同步加载"><span class="prefix"></span><span class="content">同步加载</span><span class="suffix"></span></h5>
<p>LoadingCache对象进行缓存的操作，使用CacheLoader进行缓存存储管理。批量查找可以使用getAll()方法。默认情况下，getAll()将会对缓存中没有值的key分别调用CacheLoader.load方法来构建缓存的值（build中的表达式）。我们可以重写CacheLoader.loadAll方法来提高getAll()的效率。</p>
<pre class=" language-java"><code class="prism  language-java"><span class="token comment">/*
     同步填充测试
     */</span>
    <span class="token keyword">public</span> <span class="token keyword">void</span> <span class="token function">loadingCacheTest</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token comment">//同步填充在build方法指定表达式</span>
        LoadingCache<span class="token operator">&lt;</span>String<span class="token punctuation">,</span> String<span class="token operator">&gt;</span> loadingCache <span class="token operator">=</span> Caffeine<span class="token punctuation">.</span><span class="token function">newBuilder</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">initialCapacity</span><span class="token punctuation">(</span><span class="token number">100</span><span class="token punctuation">)</span> <span class="token comment">// 初始缓存大小</span>
                <span class="token punctuation">.</span><span class="token function">maximumSize</span><span class="token punctuation">(</span><span class="token number">100</span><span class="token punctuation">)</span> <span class="token comment">// 最大缓存条数</span>
                <span class="token punctuation">.</span><span class="token function">expireAfterAccess</span><span class="token punctuation">(</span>100L<span class="token punctuation">,</span> TimeUnit<span class="token punctuation">.</span>SECONDS<span class="token punctuation">)</span> <span class="token comment">// 最后一次写入或更新固定时间过期，100秒过期</span>
                <span class="token punctuation">.</span><span class="token function">build</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token operator">:</span><span class="token operator">:</span>buildLoader<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// 构建规则，当存在没有value的key时，自动构建</span>

    <span class="token punctuation">}</span>
    <span class="token keyword">private</span> String <span class="token function">buildLoader</span><span class="token punctuation">(</span>String k<span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token keyword">return</span> k <span class="token operator">+</span> <span class="token string">"+自动补充"</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
</code></pre>
<h5 id="自动异步加载"><span class="prefix"></span><span class="content">自动异步加载</span><span class="suffix"></span></h5>
<p>AsyncLoadingCache对象进行缓存管理，get()返回一个CompletableFuture对象，默认使用ForkJoinPool.commonPool()来执行异步线程，但是我们可以通过Caffeine.executor(Executor) 方法来替换线程池。</p>
<pre class=" language-java"><code class="prism  language-java">    <span class="token comment">/**
     * 异步填充测试
     */</span>
    <span class="token keyword">public</span> <span class="token keyword">void</span> <span class="token function">asyncLoadingCacheTest</span><span class="token punctuation">(</span>String k<span class="token punctuation">)</span> <span class="token keyword">throws</span> ExecutionException<span class="token punctuation">,</span> InterruptedException <span class="token punctuation">{</span>
        <span class="token comment">//异步加载使用Executor去调用方法并返回一个CompletableFuture。异步加载缓存使用了响应式编程模型。</span>
        <span class="token comment">//</span>
        <span class="token comment">//如果要以同步方式调用时，应提供CacheLoader。要以异步表示时，应该提供一个AsyncCacheLoader，并返回一个CompletableFuture。</span>
        AsyncLoadingCache<span class="token operator">&lt;</span>String<span class="token punctuation">,</span> String<span class="token operator">&gt;</span> asyncLoadingCache <span class="token operator">=</span> Caffeine<span class="token punctuation">.</span><span class="token function">newBuilder</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">maximumSize</span><span class="token punctuation">(</span><span class="token number">100</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">expireAfterAccess</span><span class="token punctuation">(</span>100L<span class="token punctuation">,</span> TimeUnit<span class="token punctuation">.</span>SECONDS<span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">buildAsync</span><span class="token punctuation">(</span>s <span class="token operator">-</span><span class="token operator">&gt;</span> <span class="token keyword">this</span><span class="token punctuation">.</span><span class="token function">buildLoaderAsync</span><span class="token punctuation">(</span><span class="token string">"123"</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        log<span class="token punctuation">.</span><span class="token function">info</span><span class="token punctuation">(</span><span class="token string">"asyncLoadingCacheTest get： [{}] -&gt; [{}]"</span><span class="token punctuation">,</span> k<span class="token punctuation">,</span> asyncLoadingCache<span class="token punctuation">.</span><span class="token function">get</span><span class="token punctuation">(</span>k<span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token comment">//获取返回值，如果为空，则运行后面表达式，存入该缓存</span>
        log<span class="token punctuation">.</span><span class="token function">info</span><span class="token punctuation">(</span><span class="token string">"asyncLoadingCacheTest default： [{}] -&gt; [{}]"</span><span class="token punctuation">,</span> k<span class="token punctuation">,</span> asyncLoadingCache<span class="token punctuation">.</span><span class="token function">get</span><span class="token punctuation">(</span>k<span class="token punctuation">,</span> <span class="token keyword">this</span><span class="token operator">:</span><span class="token operator">:</span>buildLoader<span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>

    <span class="token keyword">private</span> CompletableFuture<span class="token operator">&lt;</span>String<span class="token operator">&gt;</span> <span class="token function">buildLoaderAsync</span><span class="token punctuation">(</span>String k<span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token keyword">return</span> CompletableFuture<span class="token punctuation">.</span><span class="token function">supplyAsync</span><span class="token punctuation">(</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">-</span><span class="token operator">&gt;</span> k <span class="token operator">+</span> <span class="token string">"+buildLoaderAsync"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
</code></pre>
<h5 id="手动异步加载"><span class="prefix"></span><span class="content">手动异步加载</span><span class="suffix"></span></h5>
<pre class=" language-java"><code class="prism  language-java">    <span class="token comment">/**
     * 手动异步测试
     */</span>
    <span class="token keyword">public</span> <span class="token keyword">void</span> <span class="token function">asyncManualCacheTest</span><span class="token punctuation">(</span>String k<span class="token punctuation">)</span> <span class="token keyword">throws</span> ExecutionException<span class="token punctuation">,</span> InterruptedException <span class="token punctuation">{</span>
        <span class="token comment">//异步加载使用Executor去调用方法并返回一个CompletableFuture。异步加载缓存使用了响应式编程模型。</span>
        <span class="token comment">//</span>
        <span class="token comment">//如果要以同步方式调用时，应提供CacheLoader。要以异步表示时，应该提供一个AsyncCacheLoader，并返回一个CompletableFuture。</span>
        AsyncCache<span class="token operator">&lt;</span>String<span class="token punctuation">,</span> String<span class="token operator">&gt;</span> asyncCache <span class="token operator">=</span> Caffeine<span class="token punctuation">.</span><span class="token function">newBuilder</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">maximumSize</span><span class="token punctuation">(</span><span class="token number">100</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">expireAfterAccess</span><span class="token punctuation">(</span>100L<span class="token punctuation">,</span> TimeUnit<span class="token punctuation">.</span>SECONDS<span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">buildAsync</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token comment">//获取返回值，如果为空，则运行后面表达式，存入该缓存</span>
        log<span class="token punctuation">.</span><span class="token function">info</span><span class="token punctuation">(</span><span class="token string">"asyncManualCacheTest default： [{}] -&gt; [{}]"</span><span class="token punctuation">,</span> k<span class="token punctuation">,</span> asyncCache<span class="token punctuation">.</span><span class="token function">get</span><span class="token punctuation">(</span>k<span class="token punctuation">,</span> <span class="token keyword">this</span><span class="token operator">:</span><span class="token operator">:</span>buildLoader<span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
</code></pre>
<h3 id="、整合springboot"><span class="prefix"></span><span class="content">3.2、整合springboot</span><span class="suffix"></span></h3>
<p>spring从3的版本开始支持Cache的处理，使用方式也是按照传统方向提供两种，基于<strong>注解</strong>和<strong>基于xml配置文件</strong>的方式，其操作纬度为方法或类，</p>
<p>比如将注解**@Cacheable**按要求配置到一个方法上，会构造出一个k-v的缓存信息，k是入参相关，value就是方法的返回值，当请求到这个方法上，先去缓存获取，有则直接返回，无则执行流程然后返回且存入缓存。还有一些其他的注解，例如：@CachePut、@CacheEvict等。</p>
<blockquote>
<p>Springboot中的spring cache是可以直接整合Caffeine cache的，配置的步骤大概</p>
</blockquote>
<ol>
<li>声明spring对cache的支持</li>
<li>方法配置缓存使用</li>
</ol>
<h4 id="依赖-1"><span class="prefix"></span><span class="content">依赖</span><span class="suffix"></span></h4>
<pre class=" language-xml"><code class="prism  language-xml"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>dependency</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>groupId</span><span class="token punctuation">&gt;</span></span>org.springframework.boot<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>groupId</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>artifactId</span><span class="token punctuation">&gt;</span></span>spring-boot-starter-cache<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>artifactId</span><span class="token punctuation">&gt;</span></span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>dependency</span><span class="token punctuation">&gt;</span></span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>dependency</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>groupId</span><span class="token punctuation">&gt;</span></span>com.github.ben-manes.caffeine<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>groupId</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>artifactId</span><span class="token punctuation">&gt;</span></span>caffeine<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>artifactId</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>version</span><span class="token punctuation">&gt;</span></span>2.6.2<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>version</span><span class="token punctuation">&gt;</span></span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>dependency</span><span class="token punctuation">&gt;</span></span>
</code></pre>
<h4 id="注解方式"><span class="prefix"></span><span class="content">注解方式</span><span class="suffix"></span></h4>
<h5 id="yml文件配置"><span class="prefix"></span><span class="content">yml文件配置</span><span class="suffix"></span></h5>
<pre class=" language-yml"><code class="prism  language-yml"><span class="token key atrule">spring</span><span class="token punctuation">:</span>
  <span class="token key atrule">cache</span><span class="token punctuation">:</span>
    <span class="token key atrule">type</span><span class="token punctuation">:</span> caffeine
    <span class="token key atrule">cache-names</span><span class="token punctuation">:</span>
      <span class="token punctuation">-</span> caffeineTestOne
      <span class="token punctuation">-</span> caffeineTestTwo
    <span class="token key atrule">caffeine</span><span class="token punctuation">:</span>
      <span class="token key atrule">spec</span><span class="token punctuation">:</span> maximumSize=500
</code></pre>
<h5 id="在启动类加注解"><span class="prefix"></span><span class="content">在启动类加注解</span><span class="suffix"></span></h5>
<blockquote>
<p>在启动类加注解：@EnableCaching，表示开始缓存功能</p>
</blockquote>
<blockquote>
<p>常用注解</p>
</blockquote>
<p>@Cacheable 触发缓存入口（这里一般放在创建和获取的方法上）</p>
<p>@CacheEvict 触发缓存的eviction（用于删除的方法上）</p>
<p>@CachePut 更新缓存且不影响方法执行（用于修改的方法上，该注解下的方法始终会被执行）</p>
<p>@Caching 将多个缓存组合在一个方法上（该注解可以允许一个方法同时设置多个注解）</p>
<p>@CacheConfig 在类级别设置一些缓存相关的共同配置（与其它缓存配合使用）</p>
<h4 id="非注解方式"><span class="prefix"></span><span class="content">非注解方式</span><span class="suffix"></span></h4>
<blockquote>
<p>在配置类初始化一个CaffeineCache对象，注册到Bean中</p>
</blockquote>
<pre class=" language-java"><code class="prism  language-java"><span class="token annotation punctuation">@Configuration</span>
<span class="token keyword">public</span> <span class="token keyword">class</span> <span class="token class-name">CacheConfig</span> <span class="token punctuation">{</span>
    <span class="token annotation punctuation">@Bean</span>
    <span class="token keyword">public</span> Cache<span class="token operator">&lt;</span>String<span class="token punctuation">,</span> Object<span class="token operator">&gt;</span> <span class="token function">caffeineCache</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token keyword">return</span> Caffeine<span class="token punctuation">.</span><span class="token function">newBuilder</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
                <span class="token comment">// 设置最后一次写入或访问后经过固定时间过期</span>
                <span class="token punctuation">.</span><span class="token function">expireAfterWrite</span><span class="token punctuation">(</span><span class="token number">60</span><span class="token punctuation">,</span> TimeUnit<span class="token punctuation">.</span>SECONDS<span class="token punctuation">)</span>
                <span class="token comment">// 初始的缓存空间大小</span>
                <span class="token punctuation">.</span><span class="token function">initialCapacity</span><span class="token punctuation">(</span><span class="token number">100</span><span class="token punctuation">)</span>
                <span class="token comment">// 缓存的最大条数</span>
                <span class="token punctuation">.</span><span class="token function">maximumSize</span><span class="token punctuation">(</span><span class="token number">1000</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">build</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<blockquote>
<p>引用</p>
</blockquote>
<pre class=" language-java"><code class="prism  language-java"><span class="token annotation punctuation">@Resource</span>
<span class="token keyword">private</span> Cache<span class="token operator">&lt;</span>String<span class="token punctuation">,</span>Object<span class="token operator">&gt;</span> cache<span class="token punctuation">;</span>
</code></pre>

