# FHIRBase Introduction

FHIRBase是用于存储和检索[FHIR 资源](http://www.hl7.org/implement/standards/fhir/resources.html)的对PostgreSQL的扩展。你可以通过任意的PostgreSQL客户端与FHIRBase进行交互操作。推荐大家使用[pgAdmin](http://www.pgadmin.org/).当然也可以选择
诸如[其他工具](https://wiki.postgresql.org/wiki/Community_Guide_to_PostgreSQL_GUI_Tools)。

通过[SQL](https://en.wikipedia.org/wiki/SQL)语言能够操作FHIRBase. 如果你不了解基本的SQL知识，请移步一些书籍或教程。

这里假设我们已经成功安装好了FHIRBase，也配置好了PostgreSQL客户端。
[FHIRBase安装指令](https://github.com/fhirbase/fhirplace#installation)

## 使用存储过程作为主要的API
在SQL的世界里，通常是使用`INSERT`来插入数据，`DELETE`来删除数据，`UPDATE`来更新数据。FHIRBase中采用了较为少用的一种方式-强制要求大家使用[存储过程](http://en.wikipedia.org/wiki/Stored_procedure)来组织管理数据。主要是为了能够实现一些FHIR特有的函数操作，诸如[search查询](http://www.hl7.org/implement/standards/fhir/search.html) 和
[versioning版本控制](http://www.hl7.org/implement/standards/fhir/http.html#vread))

在数据检索时也有一些例外.比如你可以通过  `SELECT ... FROM resource`来查询某个资源或某些资源。但当你想要新增、删除或修改时，就不得不使用我们所提到的存储过程

## 数据类型

SQL对数据类型进行严格的校验，因此，存储过程的参数和返回值都是指定了数据类型的。当我们描述存储过程时，参数和数据类型是成对出现的。比如参数`cfg` 的数据类型为 `jsonb`，写成:

<dl>
<dt>cfg (jsonb)</dt>
<dd>配置数据</dd>
</dl>
在这里可以查看完整的
[PostgreSQL数据类型概述](http://www.postgresql.org/docs/9.4/static/datatype.html#DATATYPE-TABLE).

## JSON and XML

FHIR 标准[允许](http://www.hl7.org/implement/standards/fhir/formats.html) 使用XML和JSON两种数据交换格式。
二者是可以互相转换的，也就意味着XML格式的FHIR资源可以无歧义的转换成JSON的表达格式。 FHIRBase的团队
[发布了一个XSL 2.0 转换文件](https://github.com/fhirbase/fhir-xml2json)供此之用。
鉴于XML 与 JSON 可以互相转换， FHIRBase 团队决定放弃对XML格式的支持，只支持JSON一种格式。这样做有如下优点:

* PostgreSQL本身就支持JSON数据格式，更快的查询速度和更高的存储效率;
* JSON是现如今的Web Services/APIs 流行的格式;
* 如果需要的话，可以在应用层将JSON转换成XML


## Passing JSON to a SP

当存储过程的某个参数为`jsonb`类型时, 也就是说该参数的值为JSON格式。这时候，就需要将JSON转换成单行的PostgreSQL字符串。可以有多种方法实现这个目的, 比如，使用
[在线 JSON 格式化工具](http://jsonviewer.stack.hu/). 将你的JSON数据复制到该工具中，点击 "移除空格" 即可。
另外在SQL查询中使用JSON需要对引号进行转义处理。PostgreSQL中字符串是用单引号的，比如


```sql
SELECT 'this is a string';

     ?column?
------------------
 this is a string
(1 row)
```

如果字符串中包含了单引号，必须先转为 **双引号** :

```sql
SELECT 'I''m a string with single quote!';

            ?column?
---------------------------------
 I'm a string with single quote!
(1 row)
```

因此如果你得JSON数据中包含单引号的话，在任意文本编辑器中全部将其替换成双引号。.

最后拿到的JSON, 只需要用单引号引起来，后面加上
`::jsonb`即可. 在PostgreSQL.中要使用JSON数据的话就得这么操作。

```sql
SELECT '{"foo": "i''m a string from JSON"}'::jsonb;
               jsonb
-----------------------------------
 {"foo": "i'm a string from JSON"}
(1 row)
```

## 新增、新建资源

FHIRBase安装好之后，库里面是空得，因此我们需要填充一些数据才能进行操作。

通过**fhir_create** 存储过程来新增资源，该存储过程有4个参数:

<dl>
<dt>cfg (jsonb)</dt>
<dd>配置项</dd>

<dt>resource_type (varchar)</dt>
<dd>待新增的资源类型比如. 'Organization' or 'Patient'</dd>

<dt>resource_content (jsonb)</dt>
<dd>待新增的资源内容</dd>

<dt>tags (jsonb)</dt>
<dd>每个资源所对应的 <a href="http://www.hl7.org/implement/standards/fhir/extras.html#tag">FHIR tags</a> 的数组</dd>
</dl>

**返回值 (jsonb):**
[Bundle](http://www.hl7.org/implement/standards/fhir/extras.html#bundle)，其中包含了新增的资源(这里为什么要返回bundle呢？是出于怎么样的考虑)

下面的例子会新增一个符合
[Patient resource](http://www.hl7.org/implement/standards/fhir/patient.html)
标准的患者，数据来自
[FHIR example](http://www.hl7.org/implement/standards/fhir/patient-example.json.html)，不带任何tag:

```sql
SELECT fhir_create(
  '{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  '{"resourceType":"Patient","text":{"status":"generated","div":"<div>\n      <table>\n        <tbody>\n          <tr>\n            <td>Name</td>\n            <td>Peter James <b>Chalmers</b> (&quot;Jim&quot;)</td>\n          </tr>\n          <tr>\n            <td>Address</td>\n            <td>534 Erewhon, Pleasantville, Vic, 3999</td>\n          </tr>\n          <tr>\n            <td>Contacts</td>\n            <td>Home: unknown. Work: (03) 5555 6473</td>\n          </tr>\n          <tr>\n            <td>Id</td>\n            <td>MRN: 12345 (Acme Healthcare)</td>\n          </tr>\n        </tbody>\n      </table>\n    </div>"},"identifier":[{"use":"usual","label":"MRN","system":"urn:oid:1.2.36.146.595.217.0.1","value":"12345","period":{"start":"2001-05-06"},"assigner":{"display":"Acme Healthcare"}}],"name":[{"use":"official","family":["Chalmers"],"given":["Peter","James"]},{"use":"usual","given":["Jim"]}],"telecom":[{"use":"home"},{"system":"phone","value":"(03) 5555 6473","use":"work"}],"gender":{"coding":[{"system":"http://hl7.org/fhir/v3/AdministrativeGender","code":"M","display":"Male"}]},"birthDate":"1974-12-25","deceasedBoolean":false,"address":[{"use":"home","line":["534 Erewhon St"],"city":"PleasantVille","state":"Vic","zip":"3999"}],"contact":[{"relationship":[{"coding":[{"system":"http://hl7.org/fhir/patient-contact-relationship","code":"partner"}]}],"name":{"family":["du","Marché"],"_family":[{"extension":[{"url":"http://hl7.org/fhir/Profile/iso-21090#qualifier","valueCode":"VV"}]},null],"given":["Bénédicte"]},"telecom":[{"system":"phone","value":"+33 (237) 998327"}]}],"managingOrganization":{"reference":"Organization/1"},"active":true}'::jsonb,
  '[]'::jsonb);

          fhir_create
---------------------------------------------------------------------------------
{"id": "8d33a19b-af36-4e70-ae64-e705507eb074",
"entry": [{"id": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee",
[ ... skipped ... ]
```

当新建好了资源之后，FHIRBase会为其分配一个唯一标示符。仔细查看 `fhir_create` 返回的JSON会发现其中包含2个id
第一个id是所生成的Bundle对应的id，大多数情况下并不关心该id。第二个id的路径为 `entry.0.id`, 也就是新建的Patient的ID，在后续的查询中会用到该ID

你可能会好奇为什么这个ID长得和
[URL](http://en.wikipedia.org/wiki/Uniform_resource_locator)一样? 是由于FHIR标准中资源的标识本身就是
[URL](http://www.hl7.org/implement/standards/fhir/managing.html#identity).

##  读取资源

通过**fhir_read** 存储过程来读取最新版本的资源记录:

<dl>
<dt>cfg (jsonb)</dt>
<dd>配置项</dd>

<dt>resource_type (varchar)</dt>
<dd>要读取的资源类型，如 'Organization' or 'Patient'</dd>

<dt>url (jsonb)</dt>
<dd>待读取的资源URL.</dd>

<dt>RETURNS (jsonb)</dt>
<dd>包含找到的资源的Bundle或者空bundle.</dd>
</dl>


将`[URL]`用上面所得到的患者标识替换掉，使用如下的方式调用 `fhir_read`存储过程:

```sql
SELECT fhir_read(
  '{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  '[URL]');

          fhir_read
---------------------------------------------------------------------------------
{"id": "8d33a19b-af36-4e70-ae64-e705507eb074",
"entry": [{"id": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee",
[ ... skipped ... ]
```

## 以关系型的方式读取资源内容

除了调用**fhir_read** 存储过程之外，你也 可以利用  `SELECT` 从`resource` 表中读取资源内容。这时候需要使用
[logical ID](http://www.hl7.org/implement/standards/fhir/resources.html#metadata)
和资源类型来定位资源，我们可以从资源的URL获得逻辑标识:

```
http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee
^                      ^       ^
┕ 协议 & 域名    |       ┕ 逻辑标识 ID
                       ┕ 资源类型
```

将下面的`[logical ID]` 用上面获得的患者逻辑标识来替换，执行查询即可。


```sql
SELECT content FROM resource
 WHERE logical_id = '[logical ID]'
   AND resource_type = 'Patient';

          content
---------------------------------------------------------------------------------
{"name": [{"use": "official", "given": ["Peter", "James"], "family": ["Chalmers"]},
{"use": "usual", "given": ["Jim"]}],
[... skipped ...]
```

`resource` 表中包含了FHIRBase中存储的所有资源的最新版本，但其本身并没有存储数据。
每种类型的资源都存储在自己对应类型的表中，如t: `patient`, `adversereaction`,
`encounter`,总共有48张表. 利用[PostgreSQL 表继承](http://www.postgresql.org/docs/9.4/static/ddl-inherit.html)的特性，每张表都继承了`resource` 表。

当你执行`SELECT ... FROM resource`, PostgreSQL 实际上是对每个字表进行查询，然后合并查询结果。这种方式似乎不是最搞笑的，特别是一些复杂查询，这也是使用`WHERE resource_type = '...'`的重要性，当指定了 `resource_type`, PostgreSQL 就知道具体到那张字表中进行查询。当然也可以直接对字表进行查询:

```sql
SELECT content FROM patient
 WHERE logical_id = '[logical ID]';

          content
---------------------------------------------------------------------------------
{"name": [{"use": "official", "given": ["Peter", "James"], "family": ["Chalmers"]},
{"use": "usual", "given": ["Jim"]}],
[... skipped ...]
```

一般而言，从父表`resource` 中通过逻辑标识和资源类型 `SELECT`数据 和t从字表中只通过逻辑标识SELECT数据速度一样快。


## 资源内容的更新

通过**fhir_update** 存储过程来更新资源内容:

<dl>
<dt>cfg (jsonb)</dt>
<dd>配置项</dd>

<dt>resource_type (varchar)</dt>
<dd>待更新内容的资源类型</dd>

<dt>url (varchar)</dt>
<dd>待更新内容的资源URL.</dd>

<dt>version_url (varchar)</dt>
<dd>待更新内容的资源最新版本的URL (for <a href="http://en.wikipedia.org/wiki/Optimistic_concurrency_control">optimistic locking</a>)</dd>

<dt>new_resource (jsonb)</dt>
<dd>新的资源内容</dd>

<dt>tags (jsonb)</dt>
<dd>新的tag集合.</dd>

<dt>RETURNS (jsonb)</dt>
<dd>包含了已完成内容更新的新版本资源的Bundle </dd>
</dl>

需要注意的是`version_url` 参数，在更新操作中用于[Optimistic Locking](http://en.wikipedia.org/wiki/Optimistic_concurrency_control)
一般而言，在更新之前，必须获取资源的最新版本。如果其他人在你之前同时进行了更新，你得更新操作将会失败

通过之前介绍的**fhir_read** 存储过程来读取该资源的最新版本:

```sql
SELECT fhir_read(
  '{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  '[URL]');

               fhir_read
----------------------------------------------------------------------------
{"id": "e81368f0-e353-4ca2-b9e3-bee767a187a8",
"entry": [{"id": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee",
"link": [{"rel": "self", "href": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee/_history/287cc966-8cac-4f7d-82a1-51d55a9f919e"}]
[... skipped ...]
```

 fhir_read查询结果中版本URL的路径为`entry.0.link.href`上面的例子中也就是
`http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee/_history/287cc966-8cac-4f7d-82a1-51d55a9f919e`.

然后调用`fhir_update`，将Patient.text 替换为:

```sql
SELECT fhir_update('{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  '[URL]',
  '[version URL]',
  '{"resourceType":"Patient","text":{"status":"generated","div":"UPDATED CONTENT"},"identifier":[{"use":"usual","label":"MRN","system":"urn:oid:1.2.36.146.595.217.0.1","value":"12345","period":{"start":"2001-05-06"},"assigner":{"display":"Acme Healthcare"}}],"name":[{"use":"official","family":["Chalmers"],"given":["Peter","James"]},{"use":"usual","given":["Jim"]}],"telecom":[{"use":"home"},{"system":"phone","value":"(03) 5555 6473","use":"work"}],"gender":{"coding":[{"system":"http://hl7.org/fhir/v3/AdministrativeGender","code":"M","display":"Male"}]},"birthDate":"1974-12-25","deceasedBoolean":false,"address":[{"use":"home","line":["534 Erewhon St"],"city":"PleasantVille","state":"Vic","zip":"3999"}],"contact":[{"relationship":[{"coding":[{"system":"http://hl7.org/fhir/patient-contact-relationship","code":"partner"}]}],"name":{"family":["du","Marché"],"_family":[{"extension":[{"url":"http://hl7.org/fhir/Profile/iso-21090#qualifier","valueCode":"VV"}]},null],"given":["Bénédicte"]},"telecom":[{"system":"phone","value":"+33 (237) 998327"}]}],"managingOrganization":{"reference":"Organization/1"},"active":true}'::jsonb,
  '[]'::jsonb);

                 fhir_update
-------------------------------------------------------------------------
{"id": "e4b207b8-fc2c-4317-b455-b762f26ec589", "entry": [{"id": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee", "link": [{"rel": "self", "href": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee/_history/abb33ccc-bb5a-4875-af43-9b3bba62a95c"}],
[... skipped ...]
"text": {"div": "UPDATED CONTENT", "status": "generated"}, "active": true,
[... skipped ...]
```

如果`fhir_update` 中版本URL的参数值不是最新的，将会得到如下错误信息:

```
ERROR:  Wrong version_id 43d7c2cf-a1b5-4602-b9a2-ec55d1a2dda8. Current is abb33ccc-bb5a-4875-af43-9b3bba62a95c
```

## 读取之前版本的资源内容

通过 **fhir_history** 存储过程可以读取某个资源所有的版本记录:

<dl>
<dt>cfg (jsonb)</dt>
<dd>配置项</dd>

<dt>resource_type (varchar)</dt>
<dd>资源类型.</dd>

<dt>url (varchar)</dt>
<dd>资源的URL</dd>

<dt>options (jsonb)</dt>
<dd> <a href="http://www.hl7.org/implement/standards/fhir/http.html#history">FHIR Standard for <em>history</em> RESTful action</a>.中规定的其他参数，暂未实现 .</dd>

<dt>RETURNS (jsonb)</dt>
<dd>包含了所有版本资源内容的Bundle.</dd>
</dl>

调用**fhir_history** 存储过程:

```sql
SELECT fhir_history(
  '{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  '[URL]',
  '{}'::jsonb);

                fhir_history
---------------------------------------------------------------------------
[... skipped ...]
```

通过 **fhir_vread** 存储过程来读取某个特殊版本的资源内容:

<dl>
<dt>cfg (jsonb)</dt>
<dd>配置项</dd>

<dt>resource_type (varchar)</dt>
<dd>资源类型.</dd>

<dt>version_url (varchar)</dt>
<dd>待读取的资源版本url.</dd>

<dt>RETURNS (jsonb)</dt>
<dd>包含了单个版本资源内容的Bundle </dd>
</dl>

```sql
SELECT fhir_vread(
  '{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  '[version URL]');

                fhir_vread
----------------------------------------------------------------------------
{"id": "34d2ec09-9211-4c95-a591-905279cc8212", "entry": [{"id": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee", "link": [{"rel": "self", "href": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee/_history/43d7c2cf-a1b5-4602-b9a2-ec55d1a2dda8"}],
[... skipped ...]
```

##  查询资源

[Search](http://www.hl7.org/implement/standards/fhir/search.html)是FHIR Standard中最复杂的部分. FHIRBase中实现了大部分查询功能:

*  单一查询
* 全文查询
* 根据链式变量来查询search by chainged parameters
* 根据tag来查询search by tag
* 分页和排序pagination and sorting
* 查询结果中包含其他资源including other resources in search results


这里演示如何进行简单查询，其他情况在单独成文讨论
Search 是通过**fhir_search** SP实现的:

<dl>
<dt>cfg (jsonb)</dt>
<dd>Confguration data</dd>

<dt>resource_type (varchar)</dt>
<dd>Type of resources you search for.</dd>

<dt>search_parameters (text)</dt>
<dd>Search parameters in query-string format, as described in <a href="http://www.hl7.org/implement/standards/fhir/search.html#standard">FHIR Standard</a>.</dd>

<dt>RETURNS (jsonb)</dt>
<dd>Bundle containing found resources.</dd>
</dl>

查询名字中包含"Jim"的所有患者:

```sql
SELECT fhir_search(
  '{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  'name=Jim');

                fhir_search
----------------------------------------------------------------------------
{"id": "3367b97e-4cc3-4afa-8d55-958ed686dd10", "entry": [{"id": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee",
[ ... skipped ... ]
"resourceType": "Bundle", "totalResults": 1}
```

通过MRN identifier来查询:

```sql
SELECT fhir_search(
  '{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  'identifier=urn:oid:1.2.36.146.595.217.0.1|12345');

                fhir_search
----------------------------------------------------------------------------
{"id": "3367b97e-4cc3-4afa-8d55-958ed686dd10", "entry": [{"id": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee",
[ ... skipped ... ]
"resourceType": "Bundle", "totalResults": 1}
```

通过性别来查询:

```sql
SELECT fhir_search(
  '{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  'gender=http://hl7.org/fhir/v3/AdministrativeGender|M');

                fhir_search
----------------------------------------------------------------------------
{"id": "3367b97e-4cc3-4afa-8d55-958ed686dd10", "entry": [{"id": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee",
[ ... skipped ... ]
"resourceType": "Bundle", "totalResults": 1}
```

多个查询条件的组合 :

```sql
SELECT fhir_search(
  '{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  'gender=http://hl7.org/fhir/v3/AdministrativeGender|M&name=Jim&identifier=urn:oid:1.2.36.146.595.217.0.1|12345');

                fhir_search
----------------------------------------------------------------------------
{"id": "3367b97e-4cc3-4afa-8d55-958ed686dd10", "entry": [{"id": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee",
[ ... skipped ... ]
"resourceType": "Bundle", "totalResults": 1}
```

## 删除资源

通过 **fhir_delete**来实现删除操作:

<dl>
<dt>cfg (jsonb)</dt>
<dd>Confguration data</dd>

<dt>resource_type (varchar)</dt>
<dd>Type of resources you search for.</dd>

<dt>url (varchar)</dt>
<dd>URL of resource being deleted.</dd>

<dt>RETURNS (jsonb)</dt>
<dd>Bundle containing deleted resource.</dd>
</dl>

```sql
SELECT fhir_delete(
  '{"base": "http://localhost.local"}'::jsonb,
  'Patient',
  '[URL]'
);

                fhir_delete
----------------------------------------------------------------------------
{"id": "3367b97e-4cc3-4afa-8d55-958ed686dd10", "entry": [{"id": "http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee",
[ ... skipped ... ]
"resourceType": "Bundle", "totalResults": 1}
```

NB: History of resource is also deleted(在FHIR标准中，删除操作其实是更新操作，新增一个版本的资源，并将其标记为已删除，这时候查询是查不到的，但是history操作是能罗列出的，不应该是0):

```sql
SELECT fhir_history(
  '{"base": "http://localhost.local"}'::jsonb,
  'http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee',
  '{}'::jsonb
);

                fhir_history
----------------------------------------------------------------------------
 {"id": "6d31ea93-49b3-47d2-9249-1f47d1e72c39", "entry": [], "title": "History of resource with type=http://localhost.local/Patient/b1f2890a-0536-4742-9d39-90be5d4637ee", "updated": "2014-11-25T12:42:47.634399+00:00", "resourceType": "Bundle", "totalResults": 0}
```
