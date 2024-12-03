# Access Control

The tag includes access bits that enable access control for the data stored in the tag. This chapter will explore how these access bits function. This section might feel a bit overwhelming, so I'll try to make it as simple and easy to understand as possible.

<span class="do-not-box">
⚡ Be careful when writing the access bits, as incorrect values can make the sector unusable. 
</span>

## Permissions
These are the fundamental permissions that will be used to define access conditions. The table explains each permission operation and specifies the blocks to which it is applicable: normal data blocks (read/write), value blocks, or sector trailers.

| **Operation** | **Description**                                            | **Applicable for Block Type**         |
|---------------|------------------------------------------------------------|-----------------------------------|
| **Read**      | Reads one memory block                                     | Read/Write, Value, Sector Trailer |
| **Write**     | Writes one memory block                                    | Read/Write, Value, Sector Trailer |
| **Increment** | Increments the contents of a block and stores the result in the internal Transfer Buffer | Value                             |
| **Decrement** | Decrements the contents of a block and stores the result in the internal Transfer Buffer | Value                             |
| **Restore**   | Reads the contents of a block into the internal Transfer Buffer | Value                             |
| **Transfer**  | Writes the contents of the internal Transfer Buffer to a block | Value, Read/Write                 |

## Access conditions
Let's address the elephant in the room: The access conditions. During my research, I found that many people struggled to make sense of the access condition section in the datasheet. Here is my attempt to explain it for easy to understand 🤞. 

You can use just 3 bits per block to control its permissions. In the official datasheet, this is represented using a notation like CX<sub>Y</sub> (C1₀, C1₂... C3₃) for the access bits. The first number (X) in this notation refers to the access bit number, which ranges from 1 to 3, each corresponding to a specific permission type. However, the meaning of these permissions varies depending on whether the block is a data block or a trailer block. The second number (Y) in the subscript denotes the relative block number, which ranges from 0 to 3.

### Table 1: Access conditions for the sector trailer
In the original datasheet, the subscript number is not specified in the table. I have added the subscript "3", as the sector trailer is located at Block 3.

<span class="info-box">
ℹ️ If you can read the key, it cannot be used as an authentication key. Therefore, in this table, whenever Key B is readable, it cannot serve as the authentication key. If you've noticed, yes, the Key A can never be read.
</span>
 
 <table class="table-bordered">
  <thead>
    <tr >
      <th colspan="3" rowspan="2">Access Bits</th>
      <th colspan="6">Access Condition for</th>
      <th rowspan="3">Remark</th>
    </tr>
    <tr >
      <th colspan="2">Key A</th>
      <th colspan="2">Access Bits</th>
      <th colspan="2">Key B</th>
    </tr>
    <tr >
      <th>C1<sub>3</sub></th>
      <th>C2<sub>3</sub></th>
      <th>C3<sub>3</sub></th>
      <th>Read</th>
      <th>Write</th>
      <th>Read</th>
      <th>Write</th>
      <th>Read</th>
      <th>Write</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>never</td>
      <td>key A</td>
      <td>key A</td>
      <td>never</td>
      <td>key A</td>
      <td>key A</td>
      <td>Key B may be read</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>never</td>
      <td>never</td>
      <td>key A</td>
      <td>never</td>
      <td>key A</td>
      <td>never</td>
      <td>Key B may be read</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>never</td>
      <td>key B</td>
      <td>key A|B</td>
      <td>never</td>
      <td>never</td>
      <td>key B</td>
      <td></td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>never</td>
      <td>never</td>
      <td>key A|B</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td></td>
    </tr>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>never</td>
      <td>key A</td>
      <td>key A</td>
      <td>key A</td>
      <td>key A</td>
      <td>key A</td>
      <td>Key B may be read; Default configuration</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>never</td>
      <td>key B</td>
      <td>key A|B</td>
      <td>key B</td>
      <td>never</td>
      <td>key B</td>
      <td></td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>never</td>
      <td>never</td>
      <td>key A|B</td>
      <td>key B</td>
      <td>never</td>
      <td>never</td>
      <td></td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>never</td>
      <td>never</td>
      <td>key A|B</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td></td>
    </tr>
  </tbody>
</table>

### Table 2: Access conditions for data blocks
This applies to all data blocks. The original datasheet does not include the subscript "Y", I have added it for context. Here, "Y" represents the block number (ranging from 0 to 2).

The default config here indicates that both Key A and Key B can perform all operations. However, as seen in the previous table, Key B is readable (in default config), making it unusable for authentication. Therefore, only Key A can be used.

<table class="table-bordered">
  <thead>
    <tr >
      <th colspan="3">Access Bits</th>
      <th colspan="4">Access Condition for</th>
      <th rowspan="2">Application</th>
    </tr>
    <tr >
      <th>C1<sub>Y</sub></th>
      <th>C2<sub>Y</sub></th>
      <th>C3<sub>Y</sub></th>
      <th>Read</th>
      <th>Write</th>
      <th>Increment</th>
      <th>Decrement,Transfer/Restore</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>key A|B</td>
      <td>key A|B</td>
      <td>key A|B</td>
      <td>key A|B</td>
      <td>Default configuration</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>key A|B</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td>read/write block</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>key A|B</td>
      <td>key B</td>
      <td>never</td>
      <td>never</td>
      <td>read/write block</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>key A|B</td>
      <td>key B</td>
      <td>key B</td>
      <td>key A|B</td>
      <td>value block</td>
    </tr>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>key A|B</td>
      <td>never</td>
      <td>never</td>
      <td>key A|B</td>
      <td>value block</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>key B</td>
      <td>key B</td>
      <td>never</td>
      <td>never</td>
      <td>read/write block</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>key B</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td>read/write block</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td>read/write block</td>
    </tr>
  </tbody>
</table>
[1] If key B may be read in the corresponding Sector Trailer it cannot serve for authentication (see grey marked lines in Table
7). As a consequences, if the reader authenticates any block of a sector which uses such access condition

### Table 3: Access conditions table
Let's colorize the original table to better visualize what each bit represents. The 7th and 3rd bits in each byte are related to the sector trailer. The 6th and 2nd bits correspond to Block 2. The 5th and 1st bits are associated with Block 1. The 4th and 0th bits are related to Block 0.

The overline on the notation indicates inverted values. This means that if the CX<sub>y</sub> value is 0, then <span style="text-decoration: overline;">CX</span><sub>y</sub> becomes 1.

<table >
  <thead>
    <tr>
      <th>Byte</th>
      <th>7</th>
      <th>6</th>
      <th>5</th>
      <th>4</th>
      <th>3</th>
      <th>2</th>
      <th>1</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Byte 6</td>
      <td style="background-color:#8B0000"><span style="text-decoration: overline;">C2</span><sub>3</sub></td>
      <td style="background-color:#002D62"><span style="text-decoration: overline;">C2</span><sub>2</sub></td>
      <td style="background-color:#78184A" ><span style="text-decoration: overline;">C2</span><sub>1</sub></td>
      <td style="background-color:#234F1E"><span style="text-decoration: overline;">C2</span><sub>0</sub></td>
      <td style="background-color:#8B0000"><span style="text-decoration: overline;">C1</span><sub>3</sub></td>
      <td style="background-color:#002D62"><span style="text-decoration: overline;">C1</span><sub>2</sub></td>
      <td style="background-color:#78184A"><span style="text-decoration: overline;">C1</span><sub>1</sub></td>
      <td style="background-color:#234F1E"><span style="text-decoration: overline;">C1</span><sub>0</sub></td>
    </tr>
    <tr>
      <td>Byte 7</td>
      <td style="background-color:#8B0000">C1<sub>3</sub></td>
      <td style="background-color:#002D62">C1<sub>2</sub></td>
      <td style="background-color:#78184A">C1<sub>1</sub></td>
      <td style="background-color:#234F1E">C1<sub>0</sub></td>
      <td style="background-color:#8B0000"><span style="text-decoration: overline;">C3</span><sub>3</sub></td>
      <td style="background-color:#002D62"><span style="text-decoration: overline;">C3</span><sub>2</sub></td>
      <td style="background-color:#78184A"><span style="text-decoration: overline;">C3</span><sub>1</sub></td>
      <td style="background-color:#234F1E"><span style="text-decoration: overline;">C3</span><sub>0</sub></td>
    </tr>
    <tr>
      <td>Byte 8</td>
      <td style="background-color:#8B0000">C3<sub>3</sub></td>
      <td style="background-color:#002D62">C3<sub>2</sub></td>
      <td style="background-color:#78184A">C3<sub>1</sub></td>
      <td style="background-color:#234F1E">C3<sub>0</sub></td>
      <td style="background-color:#8B0000">C2<sub>3</sub></td>
      <td style="background-color:#002D62">C2<sub>2</sub></td>
      <td style="background-color:#78184A">C2<sub>1</sub></td>
      <td style="background-color:#234F1E">C2<sub>0</sub></td>
    </tr>
  </tbody>
</table>

The default access bit "FF 07 80". Let's try to understand what it means. 
<table border="1">
  <thead>
    <tr>
      <th>Byte</th>
      <th>7</th>
      <th>6</th>
      <th>5</th>
      <th>4</th>
      <th>3</th>
      <th>2</th>
      <th>1</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Byte 6</td>
      <td style="background-color:#8B0000">1</td>
      <td style="background-color:#002D62">1</td>
      <td style="background-color:#78184A" >1</td>
      <td style="background-color:#234F1E">1</td>
      <td style="background-color:#8B0000">1</td>
      <td style="background-color:#002D62">1</td>
      <td style="background-color:#78184A">1</td>
      <td style="background-color:#234F1E">1</td>
    </tr>
    <tr>
      <td>Byte 7</td>
      <td style="background-color:#8B0000">0</td>
      <td style="background-color:#002D62">0</td>
      <td style="background-color:#78184A">0</td>
      <td style="background-color:#234F1E">0</td>
      <td style="background-color:#8B0000">0</td>
      <td style="background-color:#002D62">1</td>
      <td style="background-color:#78184A">1</td>
      <td style="background-color:#234F1E">1</td>
    </tr>
    <tr>
      <td>Byte 8</td>
      <td style="background-color:#8B0000">1</td>
      <td style="background-color:#002D62">0</td>
      <td style="background-color:#78184A">0</td>
      <td style="background-color:#234F1E">0</td>
      <td style="background-color:#8B0000">0</td>
      <td style="background-color:#002D62">0</td>
      <td style="background-color:#78184A">0</td>
      <td style="background-color:#234F1E">0</td>
    </tr>
  </tbody>
</table>

We can derive the CX<sub>Y</sub> values from the table above. Notice that only C3<sub>3</sub> is set to 1, while all other values are 0. Now, refer to Table 1 and Table 2 to understand which permission this corresponds to.
 
<table border="1">
  <thead>
    <tr>
      <th>Block</th>
      <th>C1<sub>Y</sub></th>
      <th>C2<sub>Y</sub></th>
      <th>C3<sub>Y</sub></th>
      <th>Access</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Block 0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>All permissions with Key A</td>
    </tr>
    <tr>
      <td>Block 1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>All permissions with Key A</td>
    </tr>
    <tr>
      <td>Block 2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>All permissions with Key A</td>
    </tr>
    <tr>
      <td>Block 3 (Trailer)</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>You can write Key A using Key A. Access Bits and Key B can only be read and written using Key A. </td>
    </tr>
  </tbody>
</table>

Since Key B is readable, you cannot use it for authentication. 


 ### Reference
  - [11th page of the datasheet](https://www.nxp.com/docs/en/data-sheet/MF1S50YYX_V1.pdf)
