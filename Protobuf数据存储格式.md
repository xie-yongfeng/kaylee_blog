---


---

<h2 id="一、简介"><span class="prefix"></span><span class="content">一、简介</span><span class="suffix"></span></h2>
<p>Google Protocol Buffer( 简称 Protobuf) 是 Google 公司内部的混合语言数据标准，目前已经正在使用的有超过 48,162 种报文格式定义和超过 12,183 个 .proto 文件。他们用于 RPC 系统和持续数据存储系统。</p>
<p>Protocol Buffers 是一种轻便高效的<strong>结构化数据存储格式</strong>，可以用于<strong>结构化数据串行化</strong>，或者说<strong>序列化</strong>。它很适合做数据存储或 <strong>RPC 数据交换格式</strong>。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。</p>
<h2 id="二、语法使用"><span class="prefix"></span><span class="content">二、语法使用</span><span class="suffix"></span></h2>
<h3 id="、文件头"><span class="prefix"></span><span class="content">2.1、文件头</span><span class="suffix"></span></h3>
<h4 id="syntax"><span class="prefix"></span><span class="content">syntax</span><span class="suffix"></span></h4>
<p>定义语法类型，通常proto3好于proto2，proto2好于proto1</p>
<pre class=" language-prolog"><code class="prism  language-prolog">syntax <span class="token operator">=</span> <span class="token string">"proto2"</span><span class="token operator">;</span> <span class="token operator">//</span> 定义语法类型，通常proto3好于proto2，proto2好于proto1
</code></pre>
<h4 id="import"><span class="prefix"></span><span class="content">import</span><span class="suffix"></span></h4>
<p>引入外部的组件</p>
<pre class=" language-prolog"><code class="prism  language-prolog"><span class="token operator">//</span> 引入外部的proto对象
import <span class="token string">"google/protobuf/any.proto"</span><span class="token operator">;</span>
</code></pre>
<h4 id="option-java_package"><span class="prefix"></span><span class="content">option java_package</span><span class="suffix"></span></h4>
<p>生成类的包名</p>
<pre class=" language-prolog"><code class="prism  language-prolog"><span class="token operator">//</span> 生成类的包名
option java_package <span class="token operator">=</span> "com<span class="token operator">.</span>xcyy<span class="token operator">.</span>proto<span class="token operator">;</span>
</code></pre>
<h4 id="option-java_outer_classname"><span class="prefix"></span><span class="content">option java_outer_classname</span><span class="suffix"></span></h4>
<p>生成的数据访问类的类名</p>
<pre class=" language-prolog"><code class="prism  language-prolog"><span class="token operator">//</span>生成的数据访问类的类名  
option java_outer_classname <span class="token operator">=</span> <span class="token string">"MyComplexObjectEntity"</span><span class="token operator">;</span>
</code></pre>
<h4 id="包（package）"><span class="prefix"></span><span class="content">包（package）</span><span class="suffix"></span></h4>
<p>为.proto文件添加package声明符，可以防止不同 .proto项目间消息类型的命名发生冲突。</p>
<pre class=" language-prolog"><code class="prism  language-prolog">package foo<span class="token operator">.</span>bar<span class="token operator">;</span>
</code></pre>
<h3 id="、基础数据结构"><span class="prefix"></span><span class="content">2.2、基础数据结构</span><span class="suffix"></span></h3>
<p><img src="https://img-blog.csdnimg.cn/20200419202221818.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhYWlrdWFpY2h1YW4=,size_16,color_FFFFFF,t_70" alt=""></p>
<blockquote>
<p>基本数据范围</p>
</blockquote>
<p><img src="https://pic2.zhimg.com/v2-49947fb3435b62937b098f9c9862ad2d_r.jpg" alt=""></p>
<h3 id="、消息类型（message）"><span class="prefix"></span><span class="content">2.3、消息类型（message）</span><span class="suffix"></span></h3>
<p>message用于定义结构数据，可以包含多种类型字段（field），每个字段声明以分号结尾。message经过protoc编译后会生成对应的class类，field则会生成对应的方法。</p>
<pre class=" language-prolog"><code class="prism  language-prolog">syntax <span class="token operator">=</span> <span class="token string">"proto3"</span><span class="token operator">;</span> <span class="token operator">//</span> 表示使用的protobuf版本是proto3

message <span class="token variable">SearchRequest</span> <span class="token punctuation">{</span>
  string query <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span>
  int32 page_number <span class="token operator">=</span> <span class="token number">2</span><span class="token operator">;</span>
  int32 result_per_page <span class="token operator">=</span> <span class="token number">3</span><span class="token operator">;</span>
<span class="token punctuation">}</span>
</code></pre>
<h3 id="、常规消息类型"><span class="prefix"></span><span class="content">2.4、常规消息类型</span><span class="suffix"></span></h3>
<h4 id="字段修饰符"><span class="prefix"></span><span class="content">字段修饰符</span><span class="suffix"></span></h4>
<ul>
<li>singular：单个的，有0个或1个（默认）</li>
<li>repeated：重复的，重复任意次数</li>
<li>required：要求的</li>
<li>optional：可选的</li>
<li>reserved：保留的，保留字段名或字段号</li>
</ul>
<pre class=" language-java"><code class="prism  language-java">message Foo <span class="token punctuation">{</span>
  reserved <span class="token number">2</span><span class="token punctuation">,</span> <span class="token number">15</span><span class="token punctuation">,</span> <span class="token number">9</span> to <span class="token number">11</span><span class="token punctuation">;</span>
  reserved <span class="token string">"foo"</span><span class="token punctuation">,</span> <span class="token string">"bar"</span><span class="token punctuation">;</span>
  string foo <span class="token operator">=</span> <span class="token number">3</span> <span class="token comment">// 编译报错，因为‘foo’已经被标为保留字段</span>
<span class="token punctuation">}</span>
</code></pre>
<blockquote>
<p>不能在同一个reserved语句中同时使用字段名和字段号。</p>
</blockquote>
<h4 id="枚举类型"><span class="prefix"></span><span class="content">枚举类型</span><span class="suffix"></span></h4>
<pre class=" language-prolog"><code class="prism  language-prolog">message <span class="token variable">SearchRequest</span> <span class="token punctuation">{</span>
  string query <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span>
  int32 page_number <span class="token operator">=</span> <span class="token number">2</span><span class="token operator">;</span>
  int32 result_per_page <span class="token operator">=</span> <span class="token number">3</span><span class="token operator">;</span>
  enum <span class="token variable">Corpus</span> <span class="token punctuation">{</span>
    <span class="token variable">UNIVERSAL</span> <span class="token operator">=</span> <span class="token number">0</span><span class="token operator">;</span>
    <span class="token variable">WEB</span> <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span>
    <span class="token variable">IMAGES</span> <span class="token operator">=</span> <span class="token number">2</span><span class="token operator">;</span>
    <span class="token variable">LOCAL</span> <span class="token operator">=</span> <span class="token number">3</span><span class="token operator">;</span>
    <span class="token variable">NEWS</span> <span class="token operator">=</span> <span class="token number">4</span><span class="token operator">;</span>
    <span class="token variable">PRODUCTS</span> <span class="token operator">=</span> <span class="token number">5</span><span class="token operator">;</span>
    <span class="token variable">VIDEO</span> <span class="token operator">=</span> <span class="token number">6</span><span class="token operator">;</span>
  <span class="token punctuation">}</span>
  <span class="token variable">Corpus</span> corpus <span class="token operator">=</span> <span class="token number">4</span><span class="token operator">;</span>
<span class="token punctuation">}</span>
</code></pre>
<h4 id="optional类型"><span class="prefix"></span><span class="content">optional类型</span><span class="suffix"></span></h4>
<p>消息描述中的一个元素可以被标记为“可选的”（optional）。一个格式良好的消息可以包含0个或一个optional的元素。当解 析消息时，如果它不包含optional的元素值，那么解析出来的对象中的对应字段就被置为默认值。默认值可以在消息描述文件中指定。</p>
<pre class=" language-prolog"><code class="prism  language-prolog">optional int32 result_per_page <span class="token operator">=</span> <span class="token number">3</span> <span class="token punctuation">[</span>default <span class="token operator">=</span> <span class="token number">10</span><span class="token punctuation">]</span><span class="token operator">;</span>
</code></pre>
<h4 id="map类型"><span class="prefix"></span><span class="content">map类型</span><span class="suffix"></span></h4>
<pre class=" language-prolog"><code class="prism  language-prolog">map<span class="token operator">&lt;</span>key_type<span class="token punctuation">,</span> value_type<span class="token operator">&gt;</span> map_field <span class="token operator">=</span> <span class="token variable">N</span><span class="token operator">;</span>

<span class="token operator">//</span> 与上述定义等价
message <span class="token variable">MapFieldEntry</span> <span class="token punctuation">{</span>
  key_type key <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span>
  value_type value <span class="token operator">=</span> <span class="token number">2</span><span class="token operator">;</span>
<span class="token punctuation">}</span>
repeated <span class="token variable">MapFieldEntry</span> map_field <span class="token operator">=</span> <span class="token variable">N</span><span class="token operator">;</span>
</code></pre>
<h4 id="any类型"><span class="prefix"></span><span class="content">Any类型</span><span class="suffix"></span></h4>
<p>用于定义任意的值</p>
<pre class=" language-prolog"><code class="prism  language-prolog">message <span class="token variable">ComplexObject</span> <span class="token punctuation">{</span>
   repeated google<span class="token operator">.</span>protobuf<span class="token operator">.</span><span class="token variable">Any</span> any <span class="token operator">=</span> <span class="token number">7</span><span class="token operator">;</span> <span class="token operator">//</span> <span class="token variable">Any</span>对象
<span class="token punctuation">}</span>
</code></pre>
<h4 id="oneof"><span class="prefix"></span><span class="content">oneof</span><span class="suffix"></span></h4>
<p>oneof除了共享内存中的所有字段外，oneof字段与常规字段类似，并且最多可以同时设置一个字段。</p>
<pre class=" language-prolog"><code class="prism  language-prolog">message <span class="token variable">TestMessage</span> <span class="token punctuation">{</span>
  oneof test_oneof <span class="token punctuation">{</span>
    int32 id <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span>
    string title <span class="token operator">=</span> <span class="token number">2</span><span class="token operator">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<h4 id="系统默认值"><span class="prefix"></span><span class="content">系统默认值</span><span class="suffix"></span></h4>
<ul>
<li>string默认为空字符串</li>
<li>bool默认为false</li>
<li>数值默认为0</li>
<li>enum默认为第一个元素</li>
</ul>
<h2 id="三、在java中的应用"><span class="prefix"></span><span class="content">三、在Java中的应用</span><span class="suffix"></span></h2>
<h3 id="、list"><span class="prefix"></span><span class="content">3.1、List</span><span class="suffix"></span></h3>
<pre class=" language-prolog"><code class="prism  language-prolog">message <span class="token variable">ComplexObject</span> <span class="token punctuation">{</span>
   repeated string sons <span class="token operator">=</span> <span class="token number">4</span><span class="token operator">;</span> <span class="token operator">//</span> <span class="token variable">List</span>列表
   repeated <span class="token variable">Result</span> result <span class="token operator">=</span> <span class="token number">6</span><span class="token operator">;</span> <span class="token operator">//</span> 复杂的对象<span class="token variable">List</span>
<span class="token punctuation">}</span>

<span class="token operator">//</span> 定义一个新的对象
message <span class="token variable">Result</span> <span class="token punctuation">{</span>
  string url <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span>
  string title <span class="token operator">=</span> <span class="token number">2</span><span class="token operator">;</span>
  repeated string snippets <span class="token operator">=</span> <span class="token number">3</span><span class="token operator">;</span>
<span class="token punctuation">}</span>
</code></pre>
<h3 id="、对象嵌套"><span class="prefix"></span><span class="content">3.2、对象嵌套</span><span class="suffix"></span></h3>
<pre class=" language-prolog"><code class="prism  language-prolog">message <span class="token variable">ComplexObject</span> <span class="token punctuation">{</span>
   <span class="token operator">//</span> 定义一个新的对象
  message <span class="token variable">Result</span> <span class="token punctuation">{</span>
   string url <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span>
   string title <span class="token operator">=</span> <span class="token number">2</span><span class="token operator">;</span>
   repeated string snippets <span class="token operator">=</span> <span class="token number">3</span><span class="token operator">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<h3 id="、完整proto文件"><span class="prefix"></span><span class="content">3.3、完整proto文件</span><span class="suffix"></span></h3>
<pre class=" language-prolog"><code class="prism  language-prolog"><span class="token operator">//</span> 如果使用此注释，则使用proto3<span class="token operator">;</span> 否则使用proto2
syntax <span class="token operator">=</span> <span class="token string">"proto3"</span><span class="token operator">;</span>

<span class="token operator">//</span> 引入外部的proto对象
import <span class="token string">"google/protobuf/any.proto"</span><span class="token operator">;</span>

<span class="token operator">//</span> 生成类的包名
option java_package <span class="token operator">=</span> <span class="token string">"com.hry.spring.proto.complex"</span><span class="token operator">;</span>
<span class="token operator">//</span>生成的数据访问类的类名  
option java_outer_classname <span class="token operator">=</span> <span class="token string">"MyComplexObjectEntity"</span><span class="token operator">;</span>

message <span class="token variable">ComplexObject</span> <span class="token punctuation">{</span>  
    <span class="token operator">//</span> <span class="token variable">Message</span>里每个成员变量都有一个唯一的数字标志<span class="token punctuation">(</span>   <span class="token variable">Assigning</span> <span class="token variable">Tags</span><span class="token punctuation">)</span>
    int32 id <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;//</span>  singular<span class="token punctuation">,</span> 默认值，表示成员只有<span class="token number">0</span>个或者<span class="token number">1</span>个
    string name <span class="token operator">=</span> <span class="token number">2</span><span class="token operator">;//</span> 
    string email <span class="token operator">=</span> <span class="token number">3</span><span class="token operator">;//</span>
    repeated string sons <span class="token operator">=</span> <span class="token number">4</span><span class="token operator">;</span> <span class="token operator">//</span> repeated 列表
    <span class="token variable">Gender</span> gender <span class="token operator">=</span> <span class="token number">5</span><span class="token operator">;</span> <span class="token operator">//</span> <span class="token variable">Enum</span>值
    repeated <span class="token variable">Result</span> result <span class="token operator">=</span> <span class="token number">6</span><span class="token operator">;</span> <span class="token operator">//</span> 新的对象<span class="token variable">List</span>
    repeated google<span class="token operator">.</span>protobuf<span class="token operator">.</span><span class="token variable">Any</span> any <span class="token operator">=</span> <span class="token number">7</span><span class="token operator">;</span> <span class="token operator">//</span> <span class="token variable">Any</span>对象
    map<span class="token operator">&lt;</span>string<span class="token punctuation">,</span> <span class="token variable">MapVaule</span><span class="token operator">&gt;</span> map <span class="token operator">=</span> <span class="token number">8</span><span class="token operator">;</span> <span class="token operator">//</span> 定义<span class="token variable">Map</span>对象

    <span class="token operator">//</span> reserved
    reserved <span class="token number">12</span><span class="token punctuation">,</span> <span class="token number">15</span><span class="token punctuation">,</span> <span class="token number">9</span> to <span class="token number">11</span><span class="token operator">;</span> <span class="token operator">//</span> 预留将来使用的<span class="token variable">Assigning</span> <span class="token variable">Tags</span><span class="token punctuation">,</span>
    reserved <span class="token string">"foo"</span><span class="token punctuation">,</span> <span class="token string">"bar"</span><span class="token operator">;</span> <span class="token operator">//</span> 预留将来使用的filed name
<span class="token punctuation">}</span> 

enum <span class="token variable">Gender</span> <span class="token punctuation">{</span>
  <span class="token variable">MAN</span> <span class="token operator">=</span> <span class="token number">0</span><span class="token operator">;</span>
  <span class="token variable">WOMAN</span> <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span>
<span class="token punctuation">}</span>

<span class="token operator">//</span> 定义一个新的对象
message <span class="token variable">Result</span> <span class="token punctuation">{</span>
  string url <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span>
  string title <span class="token operator">=</span> <span class="token number">2</span><span class="token operator">;</span>
  repeated string snippets <span class="token operator">=</span> <span class="token number">3</span><span class="token operator">;</span>
<span class="token punctuation">}</span>

<span class="token operator">//</span> 定义<span class="token variable">Map</span>的value值
message <span class="token variable">MapVaule</span> <span class="token punctuation">{</span>
  string mapValue <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span>
<span class="token punctuation">}</span>
</code></pre>
<h2 id="四、整合springboot项目使用proto数据传输"><span class="prefix"></span><span class="content">四、整合SpringBoot项目使用proto数据传输</span><span class="suffix"></span></h2>
<h3 id="、使用idea插件转换为.java"><span class="prefix"></span><span class="content">4.1、使用idea插件转换为.java</span><span class="suffix"></span></h3>
<h4 id="安装idea插件"><span class="prefix"></span><span class="content">安装idea插件</span><span class="suffix"></span></h4>
<p>protocol Buffers</p>
<p><img src="https://content.markdowner.net/pub/MRxQQO-zdL55ay" alt=""></p>
<h4 id="pom文件导入maven插件"><span class="prefix"></span><span class="content">pom文件导入maven插件</span><span class="suffix"></span></h4>
<pre class=" language-xml"><code class="prism  language-xml"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>build</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>extensions</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>extension</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>groupId</span><span class="token punctuation">&gt;</span></span>kr.motd.maven<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>groupId</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>artifactId</span><span class="token punctuation">&gt;</span></span>os-maven-plugin<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>artifactId</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>version</span><span class="token punctuation">&gt;</span></span>1.5.0.Final<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>version</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>extension</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>extensions</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>plugins</span><span class="token punctuation">&gt;</span></span>

            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>plugin</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>groupId</span><span class="token punctuation">&gt;</span></span>org.xolstice.maven.plugins<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>groupId</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>artifactId</span><span class="token punctuation">&gt;</span></span>protobuf-maven-plugin<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>artifactId</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>version</span><span class="token punctuation">&gt;</span></span>0.5.1<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>version</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>configuration</span><span class="token punctuation">&gt;</span></span>
                    <span class="token comment">&lt;!--suppress UnresolvedMavenProperty --&gt;</span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>protocArtifact</span><span class="token punctuation">&gt;</span></span>com.google.protobuf:protoc:3.6.1:exe:${os.detected.classifier}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>protocArtifact</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>pluginId</span><span class="token punctuation">&gt;</span></span>grpc-java<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>pluginId</span><span class="token punctuation">&gt;</span></span>
                    <span class="token comment">&lt;!--suppress UnresolvedMavenProperty --&gt;</span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>pluginArtifact</span><span class="token punctuation">&gt;</span></span>io.grpc:protoc-gen-grpc-java:1.14.0:exe:${os.detected.classifier}<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>pluginArtifact</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>configuration</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>executions</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>execution</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>goals</span><span class="token punctuation">&gt;</span></span>
                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>goal</span><span class="token punctuation">&gt;</span></span>compile<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>goal</span><span class="token punctuation">&gt;</span></span>
                            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>goal</span><span class="token punctuation">&gt;</span></span>compile-custom<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>goal</span><span class="token punctuation">&gt;</span></span>
                        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>goals</span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>execution</span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>executions</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>plugin</span><span class="token punctuation">&gt;</span></span>

        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>plugins</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>build</span><span class="token punctuation">&gt;</span></span>
</code></pre>
<h4 id="maven插件安装结果"><span class="prefix"></span><span class="content">maven插件安装结果</span><span class="suffix"></span></h4>
<p>点击右侧maven图标，在Plugins当出现protobuf时，说明安装成功</p>
<p><img src="https://content.markdowner.net/pub/B9Qrwk-waLekQe" alt=""></p>
<h4 id="编写.proto文件"><span class="prefix"></span><span class="content">编写.proto文件</span><span class="suffix"></span></h4>
<p>在<strong>src.main</strong>中新建<strong>proto</strong>文件夹</p>
<p>新建xxx.proto文件</p>
<p>如何编写.proto文件上面有源码教学</p>
<p><img src="https://content.markdowner.net/pub/e5YMzM-dRvE90B" alt=""></p>
<pre class=" language-prolog"><code class="prism  language-prolog">syntax <span class="token operator">=</span> <span class="token string">"proto3"</span><span class="token operator">;</span>  <span class="token operator">//</span>版本
option java_outer_classname <span class="token operator">=</span> <span class="token string">"GamerProto"</span><span class="token operator">;</span>    <span class="token operator">//</span>生成的外部类名，同时也是文件名

<span class="token operator">//</span>protobuf 使用massage管理数据
<span class="token operator">//</span>会在 <span class="token variable">StudentPOJO</span> 外部类生成一个内部类 <span class="token variable">Student</span>，这个才是真正发送的<span class="token variable">POJO</span>对象
message <span class="token variable">Gamer</span> <span class="token punctuation">{</span>
  int32 gamerId <span class="token operator">=</span> <span class="token number">1</span><span class="token operator">;</span> <span class="token operator">//</span><span class="token variable">Student</span>类中有一个属性名为id，类型为int32（protobuf类型），<span class="token number">1</span>表示属性序号，不是值
  string gamerName <span class="token operator">=</span> <span class="token number">2</span><span class="token operator">;</span>
  int32 gamerGrade <span class="token operator">=</span> <span class="token number">3</span><span class="token operator">;</span>
  int64 gamerCoin <span class="token operator">=</span> <span class="token number">4</span><span class="token operator">;</span>
  int32 gamerVipGrade <span class="token operator">=</span> <span class="token number">5</span><span class="token operator">;</span>
<span class="token punctuation">}</span>
</code></pre>
<h4 id="运行protobufcompile"><span class="prefix"></span><span class="content">运行protobuf:compile</span><span class="suffix"></span></h4>
<p><img src="https://content.markdowner.net/pub/zOMPvy-dArapn9" alt=""></p>
<p>运行成功</p>
<p><img src="https://content.markdowner.net/pub/B9Q3xE-Ew7N0PL" alt=""></p>
<h4 id="在tager中找到proto文件"><span class="prefix"></span><span class="content">在tager中找到proto文件</span><span class="suffix"></span></h4>
<p><img src="https://content.markdowner.net/pub/B9Q3V6-pzjPvVA" alt=""></p>
<p>将该文件移动到自己项目的<strong>src/main/com/xxx/xxx</strong>目录下</p>
<h3 id="、编写controller接口"><span class="prefix"></span><span class="content">4.3、编写controller接口</span><span class="suffix"></span></h3>
<h4 id="导入依赖"><span class="prefix"></span><span class="content">导入依赖</span><span class="suffix"></span></h4>
<pre class=" language-xml"><code class="prism  language-xml"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>dependency</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>groupId</span><span class="token punctuation">&gt;</span></span>com.google.protobuf<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>groupId</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>artifactId</span><span class="token punctuation">&gt;</span></span>protobuf-java<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>artifactId</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>version</span><span class="token punctuation">&gt;</span></span>3.6.1<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>version</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>dependency</span><span class="token punctuation">&gt;</span></span>
</code></pre>
<h4 id="编写controller接口"><span class="prefix"></span><span class="content">编写controller接口</span><span class="suffix"></span></h4>
<pre class=" language-java"><code class="prism  language-java"><span class="token annotation punctuation">@RequestMapping</span><span class="token punctuation">(</span>value <span class="token operator">=</span> <span class="token string">"/test"</span><span class="token punctuation">,</span> produces <span class="token operator">=</span> <span class="token string">"application/x-protobuf;charset=UTF-8"</span><span class="token punctuation">)</span>
    <span class="token keyword">public</span> GamerProto<span class="token punctuation">.</span>Gamer <span class="token function">getPersonProto</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        GamerProto<span class="token punctuation">.</span>Gamer<span class="token punctuation">.</span>Builder builder <span class="token operator">=</span> GamerProto<span class="token punctuation">.</span>Gamer<span class="token punctuation">.</span><span class="token function">newBuilder</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//创建proto对象</span>
        <span class="token comment">// 赋值</span>
        builder<span class="token punctuation">.</span><span class="token function">setGamerId</span><span class="token punctuation">(</span><span class="token number">2</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        builder<span class="token punctuation">.</span><span class="token function">setGamerName</span><span class="token punctuation">(</span><span class="token string">"张三"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        builder<span class="token punctuation">.</span><span class="token function">setGamerGrade</span><span class="token punctuation">(</span><span class="token number">10</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        builder<span class="token punctuation">.</span><span class="token function">setGamerCoin</span><span class="token punctuation">(</span><span class="token number">10000</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        builder<span class="token punctuation">.</span><span class="token function">setGamerVipGrade</span><span class="token punctuation">(</span><span class="token number">0</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token comment">// 返回</span>
        <span class="token keyword">return</span> builder<span class="token punctuation">.</span><span class="token function">build</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
</code></pre>
<h3 id="、测试接口"><span class="prefix"></span><span class="content">4.4、测试接口</span><span class="suffix"></span></h3>
<h4 id="使用mock进行接口访问"><span class="prefix"></span><span class="content">使用mock进行接口访问</span><span class="suffix"></span></h4>
<pre class=" language-java"><code class="prism  language-java"><span class="token comment">// 请求参数</span>
        MockHttpServletRequestBuilder getRequestBuilder <span class="token operator">=</span> MockMvcRequestBuilders
                <span class="token punctuation">.</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">"/api/gamer/proto/gamerByToken"</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">header</span><span class="token punctuation">(</span><span class="token string">"token"</span><span class="token punctuation">,</span><span class="token string">"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJnYW1lcklkIjoyfQ.Mu87PjdQS84jKrg08ayxTRHjMinBNf75NWhYiWC_GS4"</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">header</span><span class="token punctuation">(</span><span class="token string">"Content-Type"</span><span class="token punctuation">,</span> <span class="token string">"application/x-protobuf;charset=UTF-8"</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">contentType</span><span class="token punctuation">(</span><span class="token string">"application/x-protobuf;charset=UTF-8"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token comment">// 发送请求</span>
        MvcResult result <span class="token operator">=</span> mockMvc<span class="token punctuation">.</span><span class="token function">perform</span><span class="token punctuation">(</span>getRequestBuilder<span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">andDo</span><span class="token punctuation">(</span>MockMvcResultHandlers<span class="token punctuation">.</span><span class="token function">print</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
                <span class="token punctuation">.</span><span class="token function">andReturn</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token comment">// 反序列化</span>
        GamerProto<span class="token punctuation">.</span>Gamer gamerProto <span class="token operator">=</span> GamerProto<span class="token punctuation">.</span>Gamer<span class="token punctuation">.</span><span class="token function">parseFrom</span><span class="token punctuation">(</span>result<span class="token punctuation">.</span><span class="token function">getResponse</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">getContentAsByteArray</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        System<span class="token punctuation">.</span>out<span class="token punctuation">.</span><span class="token function">println</span><span class="token punctuation">(</span>gamerProto<span class="token punctuation">)</span><span class="token punctuation">;</span>



<span class="token comment">// 输出</span>
gamerId<span class="token operator">:</span> <span class="token number">2</span>
gamerName<span class="token operator">:</span> <span class="token string">"\345\274\240\344\270\211"</span>
gamerGrade<span class="token operator">:</span> <span class="token number">12</span>
gamerCoin<span class="token operator">:</span> <span class="token number">10000000</span>
gamerVipGrade<span class="token operator">:</span> <span class="token number">10</span>
</code></pre>

