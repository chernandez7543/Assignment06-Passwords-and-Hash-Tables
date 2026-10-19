# Assignment06-Passwords-and-Hash-Tables

Build a small authentication demonstration backed by your own hash table. The scenario is inspired by the password-and-hashing assignment from a prior CS 210 section.

Required implementation
Create a HashTable that stores username keys and user records.
Implement all forms of collision mitigation strategies that we looked up in class, and then choose what you find is the best mitigation strategy after implementing all of them.
Implement insert, find, remove, loadFactor, and automatic resize/rehash.
Use the following educational hash function for the password demonstration. This is for data-structures practice and is not suitable for real password security.
uint64_t demoHash(const std::string& s) {
    uint64_t h = 1469598103934665603ULL;
    for (unsigned char c : s) {
        h = (h ^ c) * 1099511628211ULL;
    }
    return h;
}
Store at least five users. Each user record must include a salt and a hash of salt + password + pepper.
Implement a login flow that looks up the username and verifies the submitted password.
Demonstrate successful login, failed password, unknown user, removal, and a rehash event.
Explain collisions, chaining, load factor, expected lookup complexity, and why a classroom hash is not a production password-hashing scheme.
Restriction: do not use std::unordered_map for the table that stores the user records.

## Hash Table Methods

| Method | How it works | Pros and Cons |
| --- | --- | --- |
| Separate chaining | Stores entries in buckets | Easy deletion; uses extra memory |
| Linear probing | `(h + i) % m` | Simple and fast; may cause clustering |
| Quadratic probing | `(h + i*i) % m` | Reduces clustering; may not check every slot |
| Double hashing | `(h + i*step) % m` | Reduces clustering; requires a second hash |

Double hashing uses `step = 1 + demoHash("step:" + username) % (m - 1)`. A prime table size helps ensure all slots can be checked.

Quadratic probing may not reach every slot, so probing methods keep the table below 50% occupied to help ensure space is available.

**Final choice: Separate chaining.** I chose this method because it makes handling collisions and deleting entries easier. It also does not require tombstones.

## Load Factor and Resizing

Load factor measures how full the hash table is:

`loadFactor() = number of entries / capacity`

- Separate chaining resizes when the load factor would exceed 0.75.
- Probing methods resize before reaching 0.50.
- Tombstones are also considered when deciding whether cleanup is needed.

For example, five entries in seven slots give a load factor of `5/7 = 0.714`. Adding a sixth entry causes the table to grow to 17 slots, giving a new load factor of `6/17 = 0.353`.

## Tests and Results

```text
PASS: separate chaining
PASS: linear probing
PASS: quadratic probing
PASS: double hashing
All tests passed.
```

The tests checked:

- Empty tables and unsuccessful searches.
- Three usernames that collide in the same bucket.
- Duplicate usernames and incorrect passwords.
- Removing and reinserting usernames.
- Tombstone handling during probing.
- Adding 100 users and resizing the table.
- Successful logins after resizing.
- Removing all users and adding a new account.
- Correct hash values and different results from different salts.

All four methods passed the tests, showing that insertion, searching, deletion, and resizing worked as expected.

## Time and Space Complexity

Let `n` represent the number of users and `m` the table capacity.

| Operation | Average Time | Worst Case |
| --- | --- | --- |
| Search | O(1) | O(n + m) |
| Insert | O(1) amortized | O(n² + m) |
| Remove | O(1) | O(n + m) |
| Rehash | O(n + m) | O(n² + m) |
| Size and load factor | O(1) | O(1) |

Average operations are fast when hashes are evenly distributed. Collisions can slow searches, and resizing requires additional work.

**Space complexity:** O(n + m) for storing users and table slots. Hashing and comparing longer strings also require additional time.

## Password Security Limitations

This project demonstrates hashing but is not intended for real password security.

- The provided hash function is fast and not cryptographically secure.
- Salts help create different hashes for identical passwords.
- The pepper is hardcoded, so it is not actually secret.
- Fixed salts make testing easier but are not secure for real accounts.

A real application should use a secure password-hashing method such as **Argon2id**, random salts, and protected secret storage.

Resources:
-	OpenAI Codex for code assistance
-	ChatGPT for troubleshooting and explanations
-	Course materials
-	YouTube: C++ Hash Tables: https://youtu.be/FsfRsGFHuv4?is=g6CbGlw9Y6DNdW7Z
  https://youtu.be/Wbdw5ucmHic?is=1RcceiCJ9v2Y73gY
