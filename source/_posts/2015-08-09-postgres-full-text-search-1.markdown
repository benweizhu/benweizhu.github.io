---
layout: post
title: "如何使用Postgres全文搜索特性（一）"
date: 2015-08-09 08:24:23 +0800
comments: true
categories: 
---

##基本概念

全文搜索提供了一种能力来鉴别的自然语言文档是否满足某个查询语句，并根据与查询语句的相关性来对查询结果排序。最常见的查询就是找到所有包含查询术语（term）的文档，并根据他们的相似性进行排序。

文字查询已经在数据库中存在多年。PostgreSQL有~，~*，LIKE和ILIKE这样的对应文字数据类型的操作符，但它们缺少现代信息系统需要的必要属性：

1.没有语言特性的支持，比如，英语中的单数和复数，用正则表达式是做不到识别复数单词的。  
2.没有查询结果的相似性的排序。   
3.因为没有索引支持，所以会比较慢

全文搜索允许对文档进行预处理，比允许建立索引来提供搜索速度。预处理一般包括：

1.将文档解析为符号。这对于解析不同类型的符号非常有用。比如，数字，单词，复杂单词，email，这样它们就可以用不同的方式被处理。原则上，符号类型取决于文档，但是大部分情况，可以使用预定义的类型就足够了。Postgres提供了一个标准的解析器，但也可以建立自定义的解析器。

2.将符号转化为分词。一个分词（词位，单词单位）是一个字符串，跟符号相似，但是它被正常化，这样就可以匹配同一个单词的不同形式，比如，正常化允许将大写字母转换为小写，更常见的是英语单词的前缀和后缀（复数单词），这就允许搜索同一个单词的不同形式，而不需要输入该单词的全部可能变形式。（话句话说，符号是文档文本的直接片段，而分词是可以用来索引和搜索的有用单词）。而Postgres实现这种方式的办法就是利用词典，可以提供不同标准的词典，或者自定义的词典来满足不同的需要。

3.Storing preprocessed documents optimized for searching. For example, each document can be represented as a sorted array of normalized lexemes. Along with the lexemes it is often desirable to store positional information to use for proximity ranking, so that a document that contains a more "dense" region of query words is assigned a higher rank than one with scattered query words.(这段没看明白)

##文档

上面提到一个很重要的概念，叫做文档（Document）。那么，它的含义是什么？

在PostgresSQL的搜索中，文档可以是数据库表中某一行的某一个文字字段（textual field），或者是多个这样的字段的组合，也许他们存储在不同表中，又或者是动态获取到的结果。话句话说，文档是由不同的部分组成，可能并不会作为一个整体存在某一个位置。比如：

{% codeblock lang:sql %}
SELECT title || ' ' ||  author || ' ' ||  abstract || ' ' || body AS document
FROM messages
WHERE mid = 12;

SELECT m.title || ' ' || m.author || ' ' || m.abstract || ' ' || d.body AS document
FROM messages m, docs d
WHERE mid = did AND mid = 12;	
{% endcodeblock %}

title，author，abstract作为了文档，它们是message的不同字段。

还有一种可能，文档以简单文本的方式存储在文件系统中，这样的情况下，数据库中可以存放全文的索引来执行搜索，并使用一些独立的标识符来从文件系统中获取文档。

##TSVECTOR（分词向量）

以文本搜索为目的，每个文档必须缩减为预处理的tsvector（某种向量）形式，搜索和排名都是在这个可以代表文档的tsvector上执行，文档只是在需要用户需要使用时才去获取，因此，我们常常将tsvector作为一种文档，当然它只是全文文档的一种压缩形式。

PostgreSQL全文搜索是基于匹配操作符@@实现，当一个tsvector（document）匹配一个tsquery（query）时，返回true。至于哪个写在前面，哪个写在后面不重要：

{% codeblock lang:sql %}
SELECT 'a fat cat sat on a mat and ate a fat rat'::tsvector @@ 'cat & rat'::tsquery;
 ?column?
----------
 t

SELECT 'fat & cow'::tsquery @@ 'a fat cat sat on a mat and ate a fat rat'::tsvector;
 ?column?
----------
 f
{% endcodeblock %}

如上面的例子中，一个tsquery不是单纯的文本，tsvector也是一样。一个tsquery包含搜索术语（term），它们是已经做过正常化的分词，并且可能通过AND，OR和NOT操作将多个术语结合。函数to_tsquery和plainto_tsquery可以帮助将用户写的文本转化为适当的tsquery。简单来说，to_tsvector是用来解析和正常化一个文档字符串的。所以，在实际应用中，一个文本搜索匹配应该长如下这样：
{% codeblock lang:sql %}
SELECT to_tsvector('fat cats ate fat rats') @@ to_tsquery('fat & rat');
 ?column? 
----------
 t
{% endcodeblock %}
注意，如果写成如下方式将不会成功：

{% codeblock lang:sql %}
SELECT 'fat cats ate fat rats'::tsvector @@ to_tsquery('fat & rat');
 ?column? 
----------
 f
{% endcodeblock %}
@@操作符也支持文本输入，简单情况下，允许跳过具体的tsvector或者tsquery的转换。使用如下：

{% codeblock lang:sql %}
tsvector @@ tsquery
tsquery  @@ tsvector
text @@ tsquery
text @@ text
{% endcodeblock %}

##如何在数据库表中进行搜索，并使用索引呢？

我们是可以再没有索引的情况下做全文搜索的，如下：

{% codeblock lang:sql %}
SELECT title
FROM pgweb
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'friend');
{% endcodeblock %}

###创建索引

我们可以创建GIN索引来提供文本搜索的速度：

{% codeblock lang:sql %}
CREATE INDEX pgweb_idx ON pgweb USING gin(to_tsvector('english', body));
{% endcodeblock %}

注意这里使用两个参数的to_tsvector方法。只有文本搜索指定了一个配置名字才能使用在表达式索引中。这是因为索引内容必须不被default_text_search_config所影响。如果它们被影响了，索引内容有可能不一致，因为不同的条目（entries）可以包含有不同配置生成的tsvector，所以没有办法猜测谁是谁。所以也就没办法正确的删除和还原这样是索引。

因为上面使用了两个参数版本的to_tsvector，只有也使用了两参数版本to_tsvector并且配置相同的查询语句才能使用。也就说，WHERE to_tsvector('english', body) @@ 'a & b'可以使用索引，WHERE to_tsvector(body) @@ 'a & b'就不行。

索引可以建立在联合的字段中：

{% codeblock lang:sql %}
CREATE INDEX pgweb_idx ON pgweb USING gin(to_tsvector('english', title || ' ' || body));
{% endcodeblock %}

###在单独的字段创建向量

另外一种办法就是创建一个单独的tsvector字段来保存to_tsvector的输出，本例中，coalesce用来保证即便有一个字段是null，仍然会生成索引：

{% codeblock lang:sql %}
ALTER TABLE pgweb ADD COLUMN textsearchable_index_col tsvector;
UPDATE pgweb SET textsearchable_index_col =
     to_tsvector('english', coalesce(title,'') || ' ' || coalesce(body,''));
{% endcodeblock %}

我们创建一个GIN索引来加快搜索：

{% codeblock lang:sql %}
CREATE INDEX textsearch_idx ON pgweb USING gin(textsearchable_index_col);
{% endcodeblock %}

现在可以来执行一个快速的全文搜索：

{% codeblock lang:sql %}
SELECT title
FROM pgweb
WHERE textsearchable_index_col @@ to_tsquery('create & table')
ORDER BY last_mod_date DESC
LIMIT 10;
{% endcodeblock %}

当使用独立字段来存储tsvector的结果时，非常有必要创建一个触发器（trigger）来保证tsvector的状态，当title或者body发生改变。

独立列的方式比表达式索引方式好的其中一点是当使用索引查询时，不用在查询语句中指定文本查询的配置。正如上面的例子中，查询语句依赖于default_text_search_config。另外一个优点是，查询更快速，因为它不需要重新执行to_tsvector来验证索引的匹配（特别是使用GiST的时候）。唯一的不同时，这种方式需要占据存储空间。

参考自：http://www.postgresql.org/docs/9.2/static/textsearch-intro.html


