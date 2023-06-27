# TVM exception codes

| Name              | Code | Definition                                                                                                                                         |
|-------------------|:----:|----------------------------------------------------------------------------------------------------------------------------------------------------|
| Stack underflow   |  2   | Not enough arguments in the stack for a primitive                                                                                                  |
| Stack overflow    |  3   | More values have been stored on a stack than allowed by this version of TVM                                                                        |
| Integer overflow  |  4   | Integer does not fit into expected range (by default −2<sup>256</sup> ≤ x < 2<sup>256</sup>), or a division by zero has occurred                   |
| Range check error |  5   | Integer out of expected range                                                                                                                      |
| Invalid opcode    |  6   | Instruction or its immediate arguments cannot be decoded                                                                                           |
| Type check error  |  7   | An argument to a primitive is of incorrect value type                                                                                              |
| Cell overflow     |  8   | Error in one of the serialization primitives                                                                                                       |
| Cell underflow    |  9   | Deserialization error                                                                                                                              |
| Dictionary error  |  10  | Error while deserializing a dictionary object                                                                                                      |
| Unknown error     |  11  | Unknown error, may be thrown by user programs                                                                                                      |
| Fatal error       |  12  | Thrown by TVM in situations deemed impossible                                                                                                      |
| Out of gas        |  13  | Thrown by TVM when the remaining gas (g r ) becomes negative. This exception usually cannot be caught and leads to an immediate termination of TVM |

See also: [TVM][1] - 4.5.7