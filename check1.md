## Check1 - Reassembler
### Some details of Reassembler

1. **TCP接收器的功能**：
   - TCP发送器会将其数据流分割成短段（每个段不超过大约1,460个字节），以确保它们可以放入数据包中。
   - 然而，网络可能会重新排序、丢弃或重复这些数据包。
   - 接收器必须将这些段重新组装成原始数据流中的连续字节序列。

2. **Reassembler接口**：
   ```cpp
   void insert( uint64_t first_index, std::string data, bool is_last_substring );
   uint64_t count_bytes_pending() const;
   Reader& reader();
   ```
3. **需求**：
   - `insert`方法告知Reassembler一个新的数据流片段，并告知其在整体流中的位置（子字符串的第一个字节的索引）。
   - Reassembler需要处理三种类型的知识：
     - 是下一个字节在流中的字节。Reassembler应尽快将这些字节推送到流中。
     - 能够容纳在流中的可用容量内的字节，但因为早期的字节未知而不能立即写入。这些字节应存储在Reassembler内部。
     - 超出流的可用容量的字节。这些字节应被丢弃。Reassembler不会存储任何无法立即或在早期字节变得已知后被推送到ByteStream的字节。
   - 处理重叠子字符串：
      - 子字符串可以重叠，但不应存储重叠的子字符串。如果提供者提供了关于同一索引的冗余信息，Reassembler应该只存储一次这种信息。
