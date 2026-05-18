# CS336-Assignment 1

# 文本编码和Tokenization

## 1.1 字符的本质

ASCII：1个字节存储127个字符 如‘A’ 对应 数字65 无法覆盖中文 会产生乱码

Unicode：有唯一ID ‘A’ 对应65 “牛”对应29275 即为 0x725B

```python
test_string = "hello! こんにちは!"
utf8_encoded = test_string.encode("utf-8")
print(utf8_encoded)
#b'hello! \xe3\x81\x93\xe3\x82\x93\xe3\x81\xab\xe3\x81\xa1\xe3\x81\xaf!'
print(type(utf8_encoded))
#<class 'bytes'>
print(list(utf8_encoded))
[104, 101, 108, 108, 111, 33, 32, 227, 129, 147, 227, 130, 147, 227, 129, 171, 227, 129, 161, 227, 129, 175, 33]
print(len(test_string))
#13
print(len(utf8_encoded))
#23
print(utf8_encoded.decode("utf-8"))
#hello! こんにちは!
```

虽然 Unicode 标准定义了字符到代码点（整数）的映射，但直接基于 Unicode 代码点训练分词器是不切实际的，因为词汇表会非常庞大（约 15 万个条目）且稀疏。因此，我们将使用 Unicode 编码，它将 Unicode 字符转换为字节序列。Unicode 标准本身定义了三种编码：UTF-8、UTF-16 和 UTF-32，其中 UTF-8 是互联网上最主要的编码（超过 98% 的网页都使用 UTF-8）。要将 Unicode 字符串编码为 UTF-8，我们可以使用 Python 中的 `encode()` 函数。要访问 Python 字节对象的底层字节值，我们可以遍历它（例如，调用 `list()` 函数）。最后，我们可以使用 `decode()` 函数将 UTF-8 字节字符串解码为 Unicode 字符串。

*Problem (unicode1): Understanding Unicode (1 point)
(a) What Unicode character does chr(0) return?
(b) How does this character’s string representation (repr()) differ from its printed representation?
(c) What happens when this character occurs in text? It may be helpful to play around with the following in your Python interpreter and see if it matches your expectations:*

![截屏2026-05-11 19.44.37.png](CS336-Assignment%201/%E6%88%AA%E5%B1%8F2026-05-11_19.44.37.png)

要将 Unicode 字符串编码为 UTF-8，我们可以使用 Python 中的 `encode()` 函数。要访问 Python 字节对象的底层字节值，我们可以遍历它（例如，调用 `list()` 函数）。最后，我们可以使用 `decode()` 函数将 UTF-8 字节字符串解码为 Unicode 字符串。

单纯用字节级（或字符级）分词虽然能解决未登录词（OOV）问题，但会导致序列过长、计算成本高且建模困难；而子词分词（subword）是在“词级”和“字节级”之间的折中，通过引入常见字节组合（如“the”）来压缩序列长度。具体做法是使用BPE（Byte Pair Encoding）算法，不断合并文本中最频繁出现的字节对，逐步构建一个子词词表，从而既减少序列长度，又保留处理新词的能力。这一构建词表的过程就称为BPE tokenizer的“训练”。

Problem (unicode2): Unicode Encodings (3 points)
(a) What are some reasons to prefer training our tokenizer on UTF-8 encoded bytes, rather than UTF-16 or UTF-32? It may be helpful to compare the output of these encodings for various input strings.
Deliverable: A one-to-two sentence response.
(b) Consider the following (incorrect) function, which is intended to decode a UTF-8 byte string into a Unicode string. Why is this function incorrect? Provide an example of an input byte string that yields incorrect results.
Deliverable: An example input byte string for which decode_utf8_bytes_to_str_wrong produces incorrect output, with a one-sentence explanation of why the function is incorrect.
(c) Give a two-byte sequence that does not decode to any Unicode character(s).
Deliverable: An example, with a one-sentence explanation.

```python
#(a) UTF-16 32 并不显著提升编码效果，且提高维度。就算是纯中文，UTF-16 也未必更好。而 UTF-32 相当于没有压缩的原始编码。
#相比于 UTF-16 或 UTF-32，UTF-8 与 ASCII 完全兼容，且在处理英文及代码等低码位字符时具有更高的空间利用率，
#能够有效减少词表体积并降低 Token 序列的平均长度。此外，UTF-8 作为变长编码不涉及字节序（Endianness）问题，在处理多语言混合文本时表现出更强的鲁棒性和普适性。
#(b)Incorrect Function:
def decode_utf8_bytes_to_str_wrong(bytestring: bytes):
  return "".join([bytes([b]).decode("utf-8") for b in bytestring])
print(decode_utf8_bytes_to_str_wrong("hello".encode("utf-8")))
#decode_utf8_bytes_to_str_wrong("世界".encode("utf-8"))
#错误原因：UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe5 in position 0: unexpected end of data
#显示 0xe4 不是一个合法的 utf-8 编码。decode_utf8_bytes_to_str_wrong的逻辑是将编码后的bytes拆成字节，再逐字节解码拼起来，但是utf-8编码是不能这么拆的
# (c) Give a two-byte sequence that does not decode to any Unicode character(s).
# Deliverable: An example, with a one-sentence explanation.
a = bytes([0b11111000, 0b00000000])
print(a)
# b'\xf8\x00'
a.decode('utf-8')
# Traceback (most recent call last):
#   File "<stdin>", line 1, in <module>
#     import platform
# UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf8 in position 0: invalid start byte
```

**UTF-8 是如何拆分字节的？（重要）**
1. 编码模版：
 

| 范围 | 字节数 | UTF-8 位模式的字节1模版(x 表示数据位) | 字节2模版 |
| --- | --- | --- | --- |
|  U+0000 ~ U+007F | 1 | 0xxxxxxx |  |
| U+0080 ~ U+07FF; | 2 | 110xxxxx  | 10xxxxxx |
| U+0800 ~ U+FFFF | 3 | 1110xxxx | 10xxxxxx |
|  U+10000 ~ U+10FFFF | 4 | 11110xxx | 10xxxxxx |

控制位：1110开头 可以判断 后面还有2个字节
延续位：10开头表示这是一个“从属字节” 并非新字节的开头

## 1.2 BPE算法及其实现

### 1.2.1 Tokenizer与Pre-tokenization

为什么需要tokenizer? 为什么不把UTF-8 直接丢给模型？
答：计算成本爆炸，注意力机制计算复杂度为O(N^2) 输入序列极长。我们需要tokenizer来平衡“词表大小”和“序列长度”。BPE是一种主流的实现方式，通过统计学手段自动寻找最值得合并的字节组合。

Extra: **tokenizers.pre_tokenizers | 预分词方法介绍**
相比于tokenizers来说，[pre_tokenizers](https://zhida.zhihu.com/search?content_id=242021404&content_type=Article&match_order=1&q=pre_tokenizers&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3Nzg2NzMzMTQsInEiOiJwcmVfdG9rZW5pemVycyIsInpoaWRhX3NvdXJjZSI6ImVudGl0eSIsImNvbnRlbnRfaWQiOjI0MjAyMTQwNCwiY29udGVudF90eXBlIjoiQXJ0aWNsZSIsIm1hdGNoX29yZGVyIjoxLCJ6ZF90b2tlbiI6bnVsbH0.LBRJ8tEbpeA8yfLUxfDZrUQ3L0emiJzkme2pFfvJEOY&zhida_source=entity)是相对而言更加简单更加容易理解的，预分词的作用，就是根据一组规则对输入的文本进行分割，这种预处理是为了确保模型不会在多个“分割”之间构建tokens。
比如如果不进行预分词，而是直接进行分词，那么可能出现这种情况："您好 人没了" -> "您" "好 人" "没了"。
也就是说分词有可能会产生这种与我们日常经验相悖的分词效果，而预分词就可以有效地避免这一点，比如在分词前，先在使用预分词在空格上进行分割："您好 人没了" -> "您好" "人没了"，再进行分词："您好" "人没了" -> "您好" "人" "没了"。
在tokenizers包中可以直接调用tokenizers.pre_tokenizers中的预分词器，如果想要调用多个，也可以使用tokenizers.pre_tokenizers.Sequence
预分词保证token完整性，学习语义。

### 1.2.2 BPE训练过程

**Step 1. Vocabulary initialization**
语料库通过utf-8转化为字节 vocab为0-255 每一个为独立token

**Step 2. Pre-tokenization 构建字典**
防止’dog.’ ‘dog!’分词不一致。字节级别的统计如‘t’和’e‘ 对于单词text 可能出现10次，我们在’t’和‘e’的频率上直接增加10。原始BPE通过去掉空格进行分词，（i.split(’ ‘)) 。GPT-2（[github.com/openai/tiktoken/pull/234/files](http://github.com/openai/tiktoken/pull/234/files)）的正则表达式为：

```python
PAT = r"""'(?:[sdmt]|ll|ve|re)| ?\p{L}+| ?\p{N}+| ?[^\s\p{L}\p{N}]+|\s+(?!\S)|\s+"""
```

统计预分词后，每个独立单位（Pre-token）出现的次数。

例：{ (b'l', b'o', b'w')：5, (b'l', b'o', b'w', b'e', b'r')：2, (b'n', b'e', b'w', b'e', b's', b't')：6 }（注意：这里的每一项都是字节 Token 的元组） 

寻找最频繁的“字节对”（Byte Pair）：扫描字典，计算所有相邻两个 Token 组合出现的总频次：(b'e', b'w') 出现了 6 次（在 newest 中）。(b'l', b'o') 出现了 5+2=7 次（在 low 和 lower 中）。为了提高 BPE 训练效率，我们不考虑跨预分词边界的字符对。在计算合并时，通过优先选择字典序较大的对来确定性地打破频率平局。

得出胜出者：(b'l', b'o') 是当前最频繁的字节对。

**Step 3：合并与记录（Merge）**
1. 产生新规则：(b'l', b'o') -> b'lo'。2. 更新词表：将新产生的 b'lo' 加入词表（分配新 ID，如 256）。3. 更新字典：将字典中所有 (b'l', b'o') 替换为 b'lo'。 迭代step2-3 直至达到词表达小预设值如10,000

性能优化：普通方法合并一个对需要遍历字典去替换，优化设计建立索引 indices[(b’l’,b’o’)] = {词1，词2} 确定合并时，只去修改词1和词2 同时更新原有邻居关系（增减计数）

**Step 4 Special Token**

<|endoftext|> (EOS) 文档结束 防止一篇末尾和另一篇开头连在一起 误认为存在因果逻辑
[PAD (padding) 用于处理batch时对齐不等长序列 如 配合attention mask 告知模型忽略此类占位符

![截屏2026-05-11 20.26.52.png](CS336-Assignment%201/%E6%88%AA%E5%B1%8F2026-05-11_20.26.52.png)

special token必须与语料库完全隔离 不能参与频率统计与合并。
防止被拆解和错误合并
实现：切分语料 独立统计 
在BPE结束后强制加入特殊token来识别并编码这些指令（通常固定ID）

Tips: Profiling
使用cProfile或py-spy等分析工具来识别实施中的瓶颈，并专注于优化这些瓶颈。
Tips: Downscaling
建议首先对数据的一小部分进行训练：“调试数据集”，而不是跳到在完整的TinyStories数据集上训练。例如，您可以在TinyStories验证集上训练您的令牌化器，该验证集是22K文档，而不是212M。

*Problem: BPE Tokenizer Training  15 points*
Write a function that, given a path to an input text file, trains a (byte-level) BPE tokenizer. Your BPE training function should handle (at least) the following input parameters.

![截屏2026-05-11 20.49.01.png](CS336-Assignment%201/%E6%88%AA%E5%B1%8F2026-05-11_20.49.01.png)

根据我们提供的测试测试您的BPE训练功能，您首先需要[adapters.run_train_bpe]中实现测试适配器。然后，运行uv run pytest tests/ test_train_bpe.py。您的实施应该能够通过所有测试。选择性实现（这可能是大量的时间投资），您可以使用一些系统语言实现训练方法的关键部分，例如C++（考虑cppyy或nanobind）或Rust（使用PyO3）。如果您这样做，请注意哪些操作需要直接从Python内存中复制与读取，并确保留下构建说明，或确保仅使用pyproject.toml构建。另请注意，GPT-2正则表达式在大多数正则表达式引擎中并不支持良好，并且在大多数正则表达式引擎中支持速度太慢。我们已经验证了Oniguruma的速度相当快，并且支持负向前看，但Python中的正则表达式包甚至更快。

作业： 见 /Users/likuntai/Desktop/assignment1-basics-main/tests/adapters.py中 run_train_pe函数和/Users/likuntai/Desktop/assignment1-basics-main/cs336_basics/train_bpe_tinystories.py

*Problem BPE Training on TinyStories (2 points)*

1.Train a byte-level BPE tokenizer on the TinyStories dataset, using a maximum vocabulary
size of 10,000. Make sure to add the TinyStories <|endoftext|> special token to the
vocabulary. Serialize the resulting vocabulary and merges to disk for further inspection. How
much time and memory did training take? What is the longest token in the vocabulary? Does
it make sense?

Hint You should be able to get under 2 minutes for BPE training using multiprocessing
during pre-tokenization and the following two facts:
(a) The <|endoftext|> token delimits documents in the data files.
(b) The <|endoftext|> token is handled as a special case before the BPE merges are applied.

2.Profile your code. What part of the tokenizer training process takes the most time?

NOTE：我们不在OpenWebText进行训练

## 1.3 BPE Tokenizer: Encoding and Decoding

在上一部分，我们实现了一个功能，在输入文本上训练BPE，以获得令牌化器词汇和BPE合并列表。现在，我们将实现一个BPE tokenizer ，该tokenizer 加载提供的词汇和合并列表，并使用它们对令牌ID的文本进行编码和解码。
我们如何将一个新句子转为Token ID？又如何将Token ID还原为句子？

### 1.3.1 Encoding text

BPE对文本进行编码的过程反映了我们如何训练BPE词汇。1.Pre-tokenize: 我们首先对序列进行预令牌化，并将每个预令牌表示为UTF-8字节序列，就像我们在BPE训练中所做的那样。我们将把每个预令牌中的这些字节合并到词汇元素中，独立处理每个预令牌（不跨预令牌边界合并）。2. Apply the merge: 然后，我们将BPE训练期间创建的词汇元素合并序列，并以相同的创建顺序将其应用。

Example (bpe_encoding): BPE encoding example

例如，假设输入字符串是 “the cat ate”词表为：{0: b' ', 1: b'a', 2: b'c', 3: b'e', 4: b'h', 5: b't', 6: b'th', 7: b' c', 8: b' a', 9: b'the', 10: b' at'}；
学习得到的 merge 规则依次为：[(b't', b'h'), (b' ', b'c'), (b' ', b'a'), (b'th', b'e'), (b' a', b't')]. 首先，预分词器会将输入切分为 “the”、“ cat” 和 “ ate” 三个 pre-token。接着，对每个 pre-token 应用 BPE merge。对于第一个 pre-token “the”，初始表示为 t、h、e 三个字节 token。检查 merge 列表后，首先发现可以应用 t 与 h 的合并，于是变成 th 和 e；随后再次检查 merge 列表，发现 th 与 e 可以继续合并，于是得到单个 token “the”；此时已经没有更多可应用的 merge，因此编码结束，对应整数序列为 9。对剩余 pre-token 重复这一过程，可以发现 “ cat” 在 merge 后表示为[b' c', b'a', b't']，对应整数序列为 [7,1,5]；最后，“ ate” 在 merge 后表示为[b' at', b'e']，对应整数序列为 10、3。因此，整个输入字符串最终被编码为整数序列：[9, 7, 1, 5, 10, 3]。

Special tokens
令牌化器在编码文本时，应有处理特殊token能力

Memory Considerations
假设我们想对一个无法容纳内存的大型文本文件进行标记。为了有效地对这个大文件（或任何其他数据流）进行令牌化，我们需要将其分解为可管理的chunk，并依次处理每个chunk，以便内存复杂性是恒定的，而不是文本大小的线性。在这样做时，我们需要确保令牌不会跨越块边界，否则我们将获得与在内存中对整个序列进行令牌化的天真方法不同的令牌化。

### 1.3.2 Decoding text

要将整数令牌ID序列解码为原始文本，我们只需在词汇（字节序列）中查找每个ID的相应条目，将它们串联在一起，然后将字节解码为Unicode字符串。
请注意，输入ID不能保证映射到有效的Unicode字符串（因为用户可以输入任何整数ID序列）。如果输入令牌ID没有产生有效的Unicode字符串，您应该用官方的Unicode替换字符U+FFFD替换格式不正确的字节。
bytes.decode的错误参数控制Unicode解码错误的处理方式，使用errors='replace'将自动用替换标记替换格式错误的数据。即: 

```python
text = byte_stream.decode('utf-8',errors = 'replace')
```

*Problem (tokenizer): Implementing the tokenizer (15 points)*
实现一个Tokenizer类，该类在给定词汇和合并列表时，将文本编码为整数ID，并将整数ID解码为文本。您的令牌化器还应该支持用户提供的特殊令牌（如果尚未存在，请将它们附在词汇中）。
To test your Tokenizer against our provided tests, you will first need to implement the test
adapter at [adapters.get_tokenizer] . Then, run uv run pytest tests/test_tokenizer.py. Your
implementation should be able to pass all tests.

# Transformer 模型架构