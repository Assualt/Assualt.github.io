---
title: avro
date: 2020-02-07 10:42:47
tags: 
- python
- c++
---

## 序列化和反序列化

### 简介
序列化：把对象转换为字节序列的过程称为对象的序列化。
反序列化：把字节序列恢复为对象的过程称为对象的反序列化。

在使用序列化的时候，通常有两个场景：

- 1）、把对象的字节序列保存在硬盘上，通常存放在一个文件中。就像打单机游戏时，现在多少级多少装备然后进行存档一样。
- 2）、在网络上传送对象的字节序列时，就要先把对象进行序列化操作，接受端接收后再进行反序列化。

一般来讲，序列化常常在java中出现，而且十分利于网络阐述。其中常见的序列化和反序列化的协议大概有如下几种：

| 方式     | 优点                                                         | 缺点                                                         | 场景                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| XML      | 1.人机可读性好 2.可指定元素或特性的名称                      | 1.序列化数据只包含数据本身以及类的结构，不包括类型标识和程序集信息 2.类必须有一个将由 XmlSerializer 序列化的默认构造函数。 3.只能序列化公共属性和字段 4.不能序列化方法 5.文件庞大，文件格式复杂，传输占带宽 | 1.当做配置文件存储数据 2.实时数据转换                        |
| JSON     | 1.前后兼容性高 2.数据格式比较简单，易于读写 3.序列化后数据较小，可扩展性好，兼容性好 4.与XML相比，其协议比较简单，解析速度比较快 | 1.数据的描述性比XML差 2.不适合性能要求为ms级别的情况 3.额外空间开销比较大 | 1.跨防火墙访问 2.可调式性要求高的情况 3.基于Web browser的Ajax请求 4.传输数据量相对小，实时性要求相对低（例如秒级别）的服务 |
| FastJson | 1.接口简单易用 2.目前java语言中最快的json库                  | 1.过于注重快，而偏离了“标准”及功能性 2.代码质量不高，文档不全 | 1.协议交互 2.Web输出 3.Android客户端                         |
| Thrift   | 1.序列化后的体积小, 速度快 2.支持多种语言和丰富的数据类型 3.对于数据字段的增删具有较强的兼容性 4.支持二进制压缩编码 | 1.使用者较少 2.跨防火墙访问时，不安全 3.不具有可读性，调试代码时相对困难 4.不能与其他传输层协议共同使用（例如HTTP） 5.无法支持向持久层直接读写数据，即不适合做数据持久化序列化协议 | 1.分布式系统的RPC解决方案                                    |
| AVRO     | 1.支持丰富的数据类型 2.简单的动态语言结合功能 3. 具有自我描述属性 4.提高了数据解析速度 5.快速可压缩的二进制数据形式 6.可以实现远程过程调用RPC 7.支持跨编程语言实现 | 1.对于习惯于静态类型语言的用户不直观                         | 1.在Hadoop中做Hive、Pig和MapReduce的持久化数据格式           |
| Protobuf | 1.序列化后码流小，性能高 2.结构化数据存储格式（XML JSON等） 3.通过标识字段的顺序，可以实现协议的前向兼容 4.结构化的文档更容易管理和维护 | 1.需要依赖于工具生成代码 2.支持的语言相对较少，官方只支持Java 、C++ 、Python | 1.对性能要求高的RPC调用 2.具有良好的跨防火墙的访问属性 3.适合应用层对象的持久化 |


## AVRO

### 简介
Avro是Hadoop的一个数据序列化系统，由Hadoop的创始人Doug Cutting（也是Lucene，Nutch等项目的创始人）开发，设计用于支持大批量数据交换的应用。
它的主要特点有：
- 支持二进制序列化方式，可以便捷，快速地处理大量数据；
- 动态语言友好，Avro提供的机制使动态语言可以方便地处理Avro数据

### 结构
- 数据它序列化和发序列化依赖Schema的数据定义实现功能。
- Avro支持两种序列化编码方式：二进制编码和JSON编码。
- Avro为了便于MapReduce的处理定义了一种容器文件格式(Container File Format)。
  - 文件中只能有一种模式，所有需要存入这个文件的对象都需要按照这种模式以二进制编码的形式写入。
  - 对象在文件中以块(Block)来组织，并且这些对象都是可以被压缩的。
  - 块和块之间会存在同步标记符(Synchronization Marker)，以便MapReduce方便地切割文件用于处理

### FastAvro
fastavro 是基于avro的基础上，底层采用c去实现底层细节，而非与avro只使用python基础类库，结果fastavro速度远快于avro。

## Avro和FastAvro性能对比(python)
fastavro能够与java的avro性能保持一个水准，而avro相较于avro要差很多。

```python
import avro.io
import avro.schema
import io
import time
import fastavro.schema
from fastavro import schemaless_writer as fwriter, schemaless_reader as freader

SCHEME = {
    "type": "record",
    "name": "Event",
    "fields": [
        {"name": "headers", "type": {"type": "map", "values": "string"}},
        {"name": "body", "type": "bytes"}
    ]
}

class FastAVROUtils(object):
    scheme = fastavro.parse_schema(SCHEME)
    
    @classmethod
    def deseriallizer(cls, message_value):
        buffer = io.BytesIO(message_value)
        # input, scheme, record to reader
        record = freader(buffer, cls.scheme)
        return record
    
    @classmethod
    def encodelizer(cls, message_value):
        buffer = io.BytesIO()
        # output, scheme, record to write
        fwriter(buffer, cls.scheme, message_value)
        return buffer.getvalue()

class AVROUtils(object):
    scheme = avro.schema.SchemaFromJSONData(SCHEME)

    @classmethod
    def deseriallizer(cls, message_value):
        buffer = io.BytesIO(message_value)
        decoder = avro.io.BinaryDecoder(buffer)
        reader = avro.io.DatumReader(cls.scheme)
        return reader.read(decoder)

    @classmethod
    def encodelizer(cls, message_value):
        buffer = io.BytesIO()
        encoder = avro.io.BinaryEncoder(buffer)
        writer = avro.io.DatumWriter(writer_schema=cls.scheme)
        writer.write(message_value, encoder)
        return buffer.getvalue()

def to_formatted_json(data):
    return {
        "headers": {
            "offset": "1234",
            "heat": "asdasdas",
            "beat.hostname": "hzs-test",
            "regionip": "123123"
        },
        "body": data
    }

def encode_to_file(filepath, encode_type = 'avro'):
    start_time = time.time() * 1000
    with open(filepath, "rb") as f1, open("encode_avro.txt", "wb") as f2:
        cnt = 0
        for d in f1.readlines():
            d = d.strip()
            if d == "":
                continue
            message = to_formatted_json(d)
            if encode_type = 'avro':
            	f2.write(AVROUtils.encodelizer(message) + bytes('\n', encoding='utf8'))
            elif encode_type = 'fastavro':
                f2.write(FastAVROUtils.encodelizer(message) + bytes('\n', encoding='utf8'))
            cnt += 1
        end_time = time.time() * 1000
        print("using {t} to encode json format message".format(t=encode_type))
        print("All {0} cost {1} ms time, average cost {2} ms time".format(cnt, end_time - start_time,
                                                                          (end_time - start_time) / cnt))
    pass

def decode_file(filepath, decode_type = 'avro'):
    with open(filepath, "rb") as f:
        cnt = 0
        start_time = time.time() * 1000
        for d in f.readlines():
            d = d.strip()
            if d == "":
                continue
            try:
                if decode_type == 'avro':
                	AVROUtils.deseriallizer(d)
                elif decode_type = 'fastavro':
                    FastAVROUtils.deseriallizer(d)
            except:
                pass
            cnt += 1
        end_time = time.time() * 1000
        print("using {t} to encode json format message".format(t=decode_type))
        print("All {0} cost {1} ms time, average cost {2} ms time".format(cnt, end_time - start_time,
                                                                          (end_time - start_time) / cnt))
        
if __name__ == '__main__':
    encode_to_file("./text.log", "avro")
    decode_file("./encode_avro.txt", "avro")
    encode_to_file("./text.log", "fastavro")
    decode_file("./encode_avro.txt", "fastavro")
```

测试结果如下：

```powershell
D:\SoftWare\python3.7\python.exe E:/Projects/python/testAvro.py
using avro to encode json format message
All 17194 Encode cost 1224.523193359375 ms time, average cost 0.07121805242290188 ms time
using avro to decode json format message
All 17196 Decode cost 1149.99609375 ms time, average cost 0.06687579051814375 ms time
using fastavro to encode json format message
All 17194 Encode cost 420.04931640625 ms time, average cost 0.024429993975005816 ms time
using fastavro to decode json format message
All 17196 Decode cost 219.99951171875 ms time, average cost 0.012793644552148755 ms time
Process finished with exit code 0
```

​	纵向对比，使用c语言作为底层实现的fastavro库明显快于只使用python作为底层实现的avro类库，而且fastavro使用相较于avro更加简单一些
[fastavro](https://github.com/fastavro/fastavro)， 官方例子附有benchmark的性能测试。从上述结果可以得出，可以尝试将avro库替换成fastavro，提高处理的效率。

参考文档：
1.https://www.perfectlyrandom.org/2019/11/29/handling-avro-files-in-python/
2.https://fastavro.readthedocs.io/en/master/reader.html