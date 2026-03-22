# URL Shortener: Review

## The Setup

Interviewer: *"Design a URL shortener."*

Me: *"Easy, just a hash table."*

**I'd be wrong.** 

Behind that simple idea lies a world of traffic estimates, hash functions, database scaling, and the epic battle of **301 vs. 302**.

Let me break down what I think the author got right, where I feel they glossed over things, and what I'd want to dig deeper into.

---

## Part 1: What I Liked — The Author Nailed These

### Smart Clarifying Questions
The chapter opens exactly how an interview should:
- 100 million new URLs/day
- 62 characters allowed (`a-z`, `A-Z`, `0-9`)
- No deletions (keep it simple)

This is how you avoid building the wrong thing.

### The Math That Matters 
```
Write: 1,160/sec | Read: 11,600/sec | 10-year storage: 365 TB
```
These numbers directly give us a **7-character** short URL (`62^7 ≈ 3.5 trillion`). I love when math drives architecture — chef's kiss.

### 301 vs. 302 — A Real Trade-off 
This was my favorite part:
- **301 (Permanent)** → Browser caches → lower server load
- **302 (Temporary)** → No caching → track every click for analytics

There's no "right" answer — it depends on whether I prioritize performance or analytics. This is what real system design feels like.

### The Hash Function Showdown 

The author compares two approaches really well:

| Approach | How It Works | Pros | Cons |
|----------|--------------|------|------|
| **Hash + Collision** | Hash URL, take 7 chars, retry on collision | No ID generator needed | Expensive DB lookups |
| **Base 62** | Convert unique ID to base-62 string | No collisions, simple | Needs distributed ID generator |

**My pick:** Base 62. I'd rather push complexity to *one component* (the ID generator) than sprinkle it throughout the flow.

---

## Part 2: Where I Started Squinting 🤔

### 1. Database Sharding — Hello? Anyone Home?
365 billion rows. The author mentions "sharding" in the wrap-up but never actually designs it.

If I were building this, I'd need to decide:

| Shard By | Good For | Problem |
|----------|----------|---------|
| `shortURL` | Fast redirects | Duplicate checks on `longURL` become expensive |
| `id` | Easy range sharding | Redirects need an index on `shortURL` |

This is a **major gap** for me. Sharding strategy affects *everything* — reads, writes, scalability.

### 2. The ID Generator — Not a Magic Box 
The author says "see Chapter 7" and moves on. But I have questions:
- If IDs are sequential, my short URLs are **predictable** — anyone could scrape my system
- If IDs are random, how do I ensure I never exceed the `62^7` limit?

This deserves more than a footnote. It's a whole design problem on its own.

### 3. Cache Strategy — What Cache?
Caching is mentioned, but with 365B URLs and 11,600 reads/sec, I'm left wondering:
- What's my cache hit ratio? (Most links are long-tail, used once or twice)
- How do I handle viral links (hot keys) that could overwhelm the cache?
- LRU? LFU? Write-through?

Caching a long tail is *hard*. I wish the author had gone deeper here.

### 4. Duplicate Check — Is This Worth It?
Step 2 in the flow: check the database to see if the long URL already exists.

With 1,160 writes/sec, that's a **massive indexed lookup** on a 365TB database. Honestly? I might just accept duplicates. Or use a Bloom filter to reduce the load.

---

## Part 3: What I'd Explore Next 

If I were actually building this system (or wanted to crush the interview), here's where I'd dig deeper:

| Topic | Why It Matters to Me |
|-------|----------------------|
| **Consistent hashing** | How I'd actually shard 365B rows across many databases |
| **Snowflake ID generator** | Distributed IDs that are time-sorted but not predictable |
| **Bloom filters** | Efficient "maybe exists" checks before hitting the DB |
| **Cache eviction policies** | LRU vs. LFU for handling both long-tail and hot keys |
| **Async analytics pipeline** | Kafka + consumers to log clicks without slowing redirects |

---

## Part 4: Brief Summary of My Critical Analysis

**What worked:** The author nails the fundamentals — smart scoping, realistic math, and clear trade-offs (especially 301 vs. 302 and the hash function showdown).

**What's missing:** Sharding strategy is underexplored, the ID generator is treated as a black box, caching lacks depth, and duplicate detection seems overkill at scale.

**Bottom line:** This is a solid foundation for a system design interview. But to build for production — or to stand out in a senior-level conversation — I'd need to fill in the gaps the author left open.

---

**Remember: if I build it, they will click.** (11,600 times per second.) 😄
