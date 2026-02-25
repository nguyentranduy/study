# Phỏng vấn kỹ thuật

Phỏng vấn kỹ thuật (Technical Interview) là vòng quan trọng nhất để đánh giá năng lực thực sự của developer.

## Các dạng phỏng vấn kỹ thuật

### 1. Coding Interview

Giải quyết bài toán coding trong thời gian giới hạn.

**Format:**

- Whiteboard coding (onsite)
- Online coding platform (HackerRank, Codility, LeetCode)
- Live coding qua screen share
- Take-home assignment

**Đánh giá:**

- Problem-solving approach
- Code quality và readability
- Time và space complexity
- Edge cases handling
- Communication

### 2. System Design Interview

Thiết kế hệ thống quy mô lớn.

**Ví dụ câu hỏi:**

- Design URL shortener (như bit.ly)
- Design Twitter/Facebook feed
- Design chat application
- Design e-commerce system

**Đánh giá:**

- Scalability thinking
- Trade-offs analysis
- Component design
- Database design

### 3. Technical Knowledge Interview

Hỏi đáp về kiến thức kỹ thuật.

**Topics:**

- Programming language (Java, Python, etc.)
- Frameworks (Spring, React, etc.)
- Database (SQL, NoSQL)
- System concepts (OS, Network)
- Best practices

---

## Coding Interview

### Approach

```
1. CLARIFY (2-3 phút)
   - Hiểu rõ yêu cầu
   - Hỏi về input/output
   - Hỏi về constraints
   - Hỏi về edge cases

2. EXAMPLES (2-3 phút)
   - Viết ra examples
   - Verify understanding

3. APPROACH (5-10 phút)
   - Brainstorm solutions
   - Discuss trade-offs
   - Choose approach
   - Explain to interviewer

4. CODE (15-20 phút)
   - Write clean code
   - Talk while coding
   - Handle edge cases

5. TEST (5 phút)
   - Walk through with example
   - Test edge cases
   - Fix bugs if any

6. OPTIMIZE (if time)
   - Discuss improvements
   - Time/space complexity
```

### Ví dụ

**Bài toán:** Tìm 2 số trong array có tổng bằng target

```java
// 1. CLARIFY
// - Input: int[] nums, int target
// - Output: indices của 2 số
// - Có duplicate không? Có
// - Luôn có solution? Giả sử có
// - Có thể dùng cùng element 2 lần? Không

// 2. EXAMPLES
// nums = [2, 7, 11, 15], target = 9
// Output: [0, 1] vì nums[0] + nums[1] = 2 + 7 = 9

// 3. APPROACH
// Brute force: O(n²) - check mọi cặp
// HashMap: O(n) - store complement

// 4. CODE
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        
        if (map.containsKey(complement)) {
            return new int[] { map.get(complement), i };
        }
        
        map.put(nums[i], i);
    }
    
    throw new IllegalArgumentException("No solution found");
}

// 5. TEST
// nums = [2, 7, 11, 15], target = 9
// i=0: complement=7, map={}, map={2:0}
// i=1: complement=2, map contains 2! return [0, 1]

// 6. OPTIMIZE
// Time: O(n), Space: O(n)
// Trade-off: space for time
```

### Common Patterns

| Pattern | Khi nào dùng | Ví dụ |
|---------|--------------|-------|
| **Two Pointers** | Sorted array, find pairs | Two Sum II, Container With Most Water |
| **Sliding Window** | Subarray/substring problems | Max Sum Subarray, Longest Substring |
| **HashMap** | Frequency, lookup | Two Sum, Group Anagrams |
| **BFS/DFS** | Graph, tree traversal | Level Order, Number of Islands |
| **Binary Search** | Sorted array, search | Search in Rotated Array |
| **Dynamic Programming** | Optimization, counting | Fibonacci, Coin Change |
| **Backtracking** | Generate combinations | Permutations, Subsets |

### Tips

!!! tip "Communication"
    - Think out loud
    - Explain approach trước khi code
    - Ask clarifying questions
    - Discuss trade-offs

!!! tip "Code Quality"
    - Use meaningful variable names
    - Write clean, readable code
    - Handle edge cases
    - Don't optimize prematurely

!!! tip "When Stuck"
    - Start with brute force
    - Think about simpler version
    - Draw examples
    - Ask for hints (it's okay!)

---

## System Design Interview

### Framework

```
1. REQUIREMENTS (5 phút)
   - Functional requirements
   - Non-functional requirements
   - Scale estimates

2. HIGH-LEVEL DESIGN (10 phút)
   - Main components
   - Data flow
   - API design

3. DEEP DIVE (20 phút)
   - Database design
   - Scaling strategies
   - Caching
   - Trade-offs

4. WRAP UP (5 phút)
   - Bottlenecks
   - Monitoring
   - Future improvements
```

### Ví dụ: Design URL Shortener

```
1. REQUIREMENTS

Functional:
- Shorten URL: long URL → short URL
- Redirect: short URL → long URL
- Custom alias (optional)
- Analytics (optional)

Non-functional:
- High availability
- Low latency (<100ms)
- Short URLs should be unique

Scale:
- 100M URLs/month = ~40 URLs/second
- Read:Write = 100:1 → 4000 reads/second
- Storage: 100M × 12 months × 500 bytes = 600GB/year

2. HIGH-LEVEL DESIGN

┌─────────┐     ┌──────────────┐     ┌──────────┐
│ Client  │────▶│ Load Balancer│────▶│ App Server│
└─────────┘     └──────────────┘     └──────────┘
                                           │
                      ┌────────────────────┼────────────────────┐
                      ▼                    ▼                    ▼
                ┌──────────┐        ┌──────────┐        ┌──────────┐
                │  Cache   │        │ Database │        │  Counter │
                │ (Redis)  │        │ (MySQL)  │        │ Service  │
                └──────────┘        └──────────┘        └──────────┘

API:
POST /api/shorten
  Request: { "long_url": "https://...", "custom_alias": "abc" }
  Response: { "short_url": "https://short.ly/abc123" }

GET /{short_code}
  Response: 301 Redirect to long_url

3. DEEP DIVE

Database Schema:
┌─────────────────────────────────────┐
│ urls                                │
├─────────────────────────────────────┤
│ id (PK)          BIGINT             │
│ short_code       VARCHAR(10) UNIQUE │
│ long_url         VARCHAR(2048)      │
│ created_at       TIMESTAMP          │
│ expires_at       TIMESTAMP          │
│ user_id          BIGINT (FK)        │
└─────────────────────────────────────┘

Short Code Generation:
- Option 1: Base62 encoding of auto-increment ID
  ID 12345 → "dnh" (base62)
- Option 2: Random string + collision check
- Option 3: Pre-generated codes in separate table

Scaling:
- Database: Read replicas, sharding by short_code
- Cache: Redis for hot URLs (LRU eviction)
- App servers: Stateless, horizontal scaling

4. WRAP UP

Bottlenecks:
- Database writes → Use write-behind cache
- Counter service → Distributed counter (Redis)

Monitoring:
- Request latency
- Cache hit rate
- Error rate
- Storage usage
```

### Key Concepts

| Concept | Giải thích |
|---------|------------|
| **Load Balancer** | Distribute traffic across servers |
| **Caching** | Store frequently accessed data in memory |
| **Database Sharding** | Split data across multiple databases |
| **Replication** | Copy data for redundancy and read scaling |
| **CDN** | Cache static content at edge locations |
| **Message Queue** | Async processing, decoupling |
| **Rate Limiting** | Prevent abuse, protect system |

---

## Technical Knowledge

### Java Questions

**OOP:**

```
Q: 4 tính chất của OOP?
A: Encapsulation, Inheritance, Polymorphism, Abstraction

Q: Abstract class vs Interface?
A: Abstract class có thể có implementation, single inheritance
   Interface chỉ có method signatures (trước Java 8), multiple inheritance
```

**Collections:**

```
Q: ArrayList vs LinkedList?
A: ArrayList: O(1) random access, O(n) insert/delete
   LinkedList: O(n) random access, O(1) insert/delete at ends

Q: HashMap internal working?
A: Array of buckets, hash function determines bucket
   Collision handling: linked list (Java 7) or tree (Java 8+)
```

**Multithreading:**

```
Q: synchronized vs volatile?
A: synchronized: mutual exclusion, visibility
   volatile: visibility only, no atomicity

Q: Thread pool benefits?
A: Reuse threads, control concurrency, queue management
```

### Spring Questions

```
Q: Dependency Injection là gì?
A: Design pattern where dependencies are provided externally
   rather than created inside the class. Promotes loose coupling.

Q: @Component vs @Service vs @Repository?
A: All are stereotypes for component scanning
   @Service: business logic layer
   @Repository: data access layer, exception translation

Q: Bean scopes?
A: singleton (default), prototype, request, session, application
```

### Database Questions

```
Q: Index là gì? Khi nào dùng?
A: Data structure to speed up queries
   Use for: WHERE, JOIN, ORDER BY columns
   Don't use for: small tables, frequently updated columns

Q: ACID properties?
A: Atomicity: all or nothing
   Consistency: valid state to valid state
   Isolation: concurrent transactions don't interfere
   Durability: committed data persists

Q: SQL vs NoSQL?
A: SQL: structured, ACID, relationships
   NoSQL: flexible schema, horizontal scaling, eventual consistency
```

---

## Practice Resources

### Coding

| Platform | Đặc điểm |
|----------|----------|
| [LeetCode](https://leetcode.com) | Comprehensive, company-tagged |
| [HackerRank](https://hackerrank.com) | Good for beginners |
| [Codility](https://codility.com) | Used by many companies |
| [AlgoExpert](https://algoexpert.io) | Video explanations |

### System Design

| Resource | Đặc điểm |
|----------|----------|
| [System Design Primer](https://github.com/donnemartin/system-design-primer) | Free, comprehensive |
| [Grokking System Design](https://www.educative.io/courses/grokking-the-system-design-interview) | Structured course |
| [ByteByteGo](https://bytebytego.com) | Visual explanations |

### Mock Interviews

| Platform | Đặc điểm |
|----------|----------|
| [Pramp](https://pramp.com) | Free peer interviews |
| [Interviewing.io](https://interviewing.io) | Anonymous, with engineers |

---

## Checklist

### Trước phỏng vấn

- [ ] Practice 50-100 LeetCode problems
- [ ] Review data structures và algorithms
- [ ] Study system design basics
- [ ] Review language/framework knowledge
- [ ] Do mock interviews

### Trong phỏng vấn

- [ ] Clarify requirements
- [ ] Think out loud
- [ ] Start with brute force
- [ ] Write clean code
- [ ] Test with examples
- [ ] Discuss complexity

### Sau phỏng vấn

- [ ] Review problems đã gặp
- [ ] Practice similar problems
- [ ] Note areas to improve

## Tiếp theo

- [Làm việc nhóm](../work/teamwork.md)
