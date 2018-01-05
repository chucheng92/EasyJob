## URI和URL的区别

这两天在写代码的时候，由于涉及到资源的位置，因此，需要在Java Bean中定义一些字段，用来表示资源的位置，比如：imgUrl，logoUri等等。但是，每次定义的时候，心里都很纠结，是该用imgUrl还是imgUri呢？

同样的，另外一个问题：String HttpServletRequest.getRequestURI()；和StringBuffer HttpServletRequest.getRequestURL();返回的内容有何不同？为什么会如此？

带着这些问题到网上去搜了下，没发现让自己看了明白的解释，于是，想到了Java类库里有两个对应的类java.net.URI和java.net.URL，终于，在这两个类里的javadoc里找到了答案。

**URIs, URLs, and URNs**

首先，URI，是uniform resource identifier，统一资源标识符，用来唯一的标识一个资源。而URL是uniform resource locator，统一资源定位器，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。而URN，uniform resource name，统一资源命名，是通过名字来标识资源，比如mailto:java-net@java.sun.com。也就是说，URI是以一种抽象的，高层次概念定义统一资源标识，而URL和URN则是具体的资源标识的方式。URL和URN都是一种URI。

在Java的URI中，一个URI实例可以代表绝对的，也可以是相对的，只要它符合URI的语法规则。而URL类则不仅符合语义，还包含了定位该资源的信息，因此它不能是相对的，schema必须被指定。

ok，现在回答文章开头提出的问题，到底是imgUrl好呢，还是imgUri好？显然，如果说imgUri是肯定没问题的，因为即使它实际上是url，那它也是uri的一种。那么用imgUrl有没有问题呢？此时则要看它的可能取值，如果是绝对路径，能够定位的，那么用imgUrl是没问题的，而如果是相对路径，那还是不要用ImgUrl的好。总之，用imgUri是肯定没问题的，而用imgUrl则要视实际情况而定。

第二个，从HttpServletRequest的javadoc中可以看出，getRequestURI返回一个String，“the part of this request’s URL from the protocol name up to the query string in the first line of the HTTP request”，比如“POST /some/path.html?a=b HTTP/1.1”，则返回的值为”/some/path.html”。现在可以明白为什么是getRequestURI而不是getRequestURL了，因为此处返回的是相对的路径。而getRequestURL返回一个StringBuffer，“The returned URL contains a protocol, server name, port number, and server path, but it does not include query string parameters.”，完整的请求资源路径，不包括querystring。

总结一下：URL是一种具体的URI，它不仅唯一标识资源，而且还提供了定位该资源的信息。URI是一种语义上的抽象概念，可以是绝对的，也可以是相对的，而URL则必须提供足够的信息来定位，所以，是绝对的，而通常说的relative URL，则是针对另一个absolute URL，本质上还是绝对的。

注：这里的绝对(absolute)是指包含scheme，而相对(relative)则不包含scheme。

URI抽象结构     [scheme:]scheme-specific-part[#fragment]

[scheme:][//authority][path][?query][#fragment]

authority为[user-info@]host[:port]

参考资料：

http://docs.oracle.com/javase/1.5.0/docs/api/java/net/URI.html

http://en.wikipedia.org/wiki/Uniform_Resource_Identifier

http://docs.oracle.com/javaee/5/api/javax/servlet/http/HttpServletRequest.html

ps:

java.net.URL类不提供对标准RFC2396规定的特殊字符的转义，因此需要调用者自己对URL各组成部分进行encode。而java.net.URI则会提供转义功能。因此The recommended way  to manage the encoding and decoding of URLs is to use  java.net.URI. 可以使用URI.toURL()和URL.toURI()方法来对两个类型的对象互相转换。对于HTML FORM的url encode/decode可以使用java.net.URLEncoder和java.net.URLDecoder来完成，但是对URL对象不适用。