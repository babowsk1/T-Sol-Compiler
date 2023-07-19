# String

T-Sol Compiler expands `string` type with the following functions:

**Note**: Due to VM restrictions string length can't exceed `1024 * 127 = 130048` bytes.

## empty()

```solidity
<string>.empty() returns (bool);
```

Returns status flag whether the `string` is empty (its length is 0).

## byteLength()

```solidity
<string>.byteLength() returns (uint32);
```

Returns byte length of the `string` data.

## substr()

```solidity
<string>.substr(uint from[, uint count]) returns (string);
```

Returns the substring starting from the byte with number **from** with byte length **count**.\
**Note**: if count is not set, then the new `string` will be cut from the **from** byte to the end of the string.

```solidity
string long = "0123456789";
string a = long.substr(1, 2); // a = "12"
string b = long.substr(6); // b = "6789"
```

## append()

```solidity
<string>.append(string tail);
```

Appends the tail `string` to the `string`.

## operator+

```solidity
<string>.operator+(string) returns (string);
<string>.operator+(bytesN) returns (string);
```

Creates new `string` as a concatenation of left and right arguments of the operator +. Example:

```solidity
string a = "abc";
bytes2 b = "12";
string c = a + b; // "abc12"
```

## find() and findLast()

```solidity
<string>.find(bytes1 symbol) returns (optional(uint32));
<string>.find(string substr) returns (optional(uint32));
<string>.findLast(bytes1 symbol) returns (optional(uint32));
```

Looks for **symbol** (or substring) in the `string` and returns index of the first (`find`) or the last (`findLast`) occurrence. If there is no such symbol in the `string`, an empty optional is returned.

Example:

```solidity
string str = "01234567890";
optional(uint32) a = str.find(byte('0'));
bool s = a.hasValue(); // s == true
uint32 n = a.get(); // n == 0

byte symbol = 'a';
optional(uint32) b = str.findLast(symbol);
bool s = b.hasValue(); // s == false

string sub = "111";
optional(uint32) c = str.find(sub);
bool s = c.hasValue(); // s == false
```

Same as [\<TvmCell>.dataSizeQ()](string.md#datasizeq).

{% hint style="warning" %}
In version 0.67.0, new functions were added to this method.
For more details, please refer to the corresponding version of the documentation.
{% endhint %}


## toUpperCase()\` and toLowerCase()

```solidity
<string>.toUpperCase() returns (string)
<string>.toLowerCase() returns (string)
```

Converts `string` to upper/lower case. It treats `string` as ASCII string and converts only 'A'..'Z' and 'a'..'z' symbols. Example:

```solidity
string s = "Hello";
string a = s.toUpperCase(); // a == "HELLO" 
string b = s.toLowerCase(); // b == "hello" 
```

## format()

```solidity
format(string template, TypeA a, TypeB b, ...) returns (string);
```

Builds a `string` with arbitrary parameters. Empty placeholder {} can be filled with integer (in decimal view), address, string or fixed point number. Fixed point number is printed based on its type (`fixedMxN` is printed with `N` digits after dot). Placeholder should be specified in such formats:

* `"{}"` - empty placeholder
* `"{:[0]<width>{"x","d","X","t"}}"` - placeholder for integers. Fills num with 0 if format starts with "0". Formats integer to have specified `width`. Can format integers in decimal ("d" postfix), lower hex ("x") or upper hex ("X") form. Format "t" prints number (in nanotons) as a fixed point Ton sum.

Warning: this function consumes too much gas, that's why it's better not to use it onchain. Example:

```solidity
string str = format("Hello {} 0x{:X} {}  {}.{} tons", 123, 255, address.makeAddrStd(-33,0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF123456789ABCDE), 100500, 32);
// str == "Hello 123 0xFF -21:7fffffffffffffffffffffffffffffffffffffffffffffffff123456789abcde  100500.32 tons"
str = format("Hello {}", 123); // str == "Hello 123"
str = format("Hello 0x{:X}", 123); // str == "Hello 0x7B"
str = format("{}", -123); // str == "-123"
str = format("{}", address.makeAddrStd(127,0)); // str == "7f:0000000000000000000000000000000000000000000000000000000000000000"
str = format("{}", address.makeAddrStd(-128,0)); // str == "-80:0000000000000000000000000000000000000000000000000000000000000000"
str = format("{:6}", 123); // str == "   123"
str = format("{:06}", 123); // str == "000123"
str = format("{:06d}", 123); // str == "000123"
str = format("{:06X}", 123); // str == "00007B"
str = format("{:6x}", 123); // str == "    7b"
uint128 a = 1 ton;
str = format("{:t}", a); // str == "1.000000000"
a = 123;
str = format("{:t}", a); // str == "0.000000123"
fixed32x3 v = 1.5;
str = format("{}", v); // str == "1.500"
fixed256x10 vv = -987123.4567890321;
str = format("{}", vv); // str == "-987123.4567890321"
```

## stoi()

```solidity
stoi(string inputStr) returns (optional(int) /*result*/);
```

Converts a `string` into an integer. If `string` starts with '0x' it will be converted from a hexadecimal format, otherwise it is meant to be number in decimal format. Function returns the integer in case of success. Otherwise, returns `null`.

Warning: this function consumes too much gas, that's why it's better not to use it onchain. Example:

```solidity
optional(int) res;
res = stoi("123"); // res ==123
string hexstr = "0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF123456789ABCDE";
res = stoi(hexstr); // res == 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF123456789ABCDE
res = stoi("0xag"); // res == null
```

## string conversion

```solidity
string s = "1";
bytes2 b = bytes2(s); // b == 0x3100

string s = "11";
bytes2 b = bytes2(s); // b = 0x3131
```

`string` can be converted to `bytesN` which causes **N** \* 8 bits being loaded from the cell and saved to variable. If `string` object has less than **N** bytes, extra bytes are padded with zero bits.
