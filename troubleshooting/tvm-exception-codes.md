# TVM exception codes

Tvm exception code can differ in different networks:
for C++ nodes networks they should correspond original documentation
([TVM](https://broxus.gitbook.io/threaded-virtual-machine/)- 4.5.7);
for Rust nodes network codes are converted to other values:
 `rust_code = -(c++_code + 1)`

List of codes:

| Name              | C++ code | Rust code | Definition                                                                                                                                         |
|-------------------|:--------:|:---------:|----------------------------------------------------------------------------------------------------------------------------------------------------|
| Stack underflow   |    2     |    -3     | Not enough arguments in the stack for a primitive                                                                                                  |
| Stack overflow    |    3     |    -4     | More values have been stored on a stack than allowed by this version of TVM                                                                        |
| Integer overflow  |    4     |    -5     | Integer does not fit into expected range (by default −2<sup>256</sup> ≤ x < 2<sup>256</sup>), or a division by zero has occurred                   |
| Range check error |    5     |    -6     | Integer out of expected range                                                                                                                      |
| Invalid opcode    |    6     |    -7     | Instruction or its immediate arguments cannot be decoded                                                                                           |
| Type check error  |    7     |    -8     | An argument to a primitive is of incorrect value type                                                                                              |
| Cell overflow     |    8     |    -9     | Error in one of the serialization primitives                                                                                                       |
| Cell underflow    |    9     |    -10    | Deserialization error                                                                                                                              |
| Dictionary error  |    10    |    -11    | Error while deserializing a dictionary object                                                                                                      |
| Unknown error     |    11    |    -12    | Unknown error, may be thrown by user programs                                                                                                      |
| Fatal error       |    12    |    -13    | Thrown by TVM in situations deemed impossible                                                                                                      |
| Out of gas        |    13    |    -14    | Thrown by TVM when the remaining gas (g r ) becomes negative. This exception usually cannot be caught and leads to an immediate termination of TVM |