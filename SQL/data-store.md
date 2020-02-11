 ### 数值类型存储要求

| Data Type                    | Storage Required            |
| ---------------------------- | --------------------------- |
| tinyint, smallint, mediumint | 1 B, 2 B, 3 B               |
| int, integer                 | 4 B                         |
| bigint                       | 8 B                         |
| float                        | 4 B                         |
| double, real                 | 8 B                         |
| decimal(M, D), numeric(M, D) | <= 4 + 4 B                  |
| bit(M)                       | approximately (M + 7) / 8 B |

### 字符串类型存储要求

M: declared column length

L: actual column length

unit: B

| Data Type                | Storage Required                   |
| ------------------------ | ---------------------------------- |
| char(M)                  |                                    |
| binary(M)                | M (0 <= M <= 255)                  |
| varchar(M), varbinary(M) | L + 1 (0<=M<=255), L + 2 (M > 255) |
| tinyblob, tinytext       | L + 1 (L < 2^ 8)                   |
| blob, text               | L + 2                              |
| mediumblob, mediumtext   | L + 3                              |
| longblob, longtext       | L + 4                              |

