## Huffman Coding

**Topic** – Greedy Algorithm / Data Compression / Optimal Prefix‑Free Codes

#### Standard version (Min‑Heap / Priority Queue based)

```cpp
#include <vector>
#include <queue>
#include <string>
#include <unordered_map>

using namespace std;

struct HuffNode {
    char ch;        // character ('\0' for internal nodes)
    int freq;       // frequency of character or sum of children
    HuffNode* left;
    HuffNode* right;

    HuffNode(char c, int f) : ch(c), freq(f), left(nullptr), right(nullptr) {}
};

// Comparator for min‑heap: lower frequency has higher priority
struct Compare {
    bool operator()(HuffNode* a, HuffNode* b) {
        return a->freq > b->freq;
    }
};

// Build the Huffman tree and return its root
HuffNode* buildHuffmanTree(const unordered_map<char, int>& freqMap) {
    // Step 1: Create a leaf node for each character and push into min‑heap
    priority_queue<HuffNode*, vector<HuffNode*>, Compare> pq;
    for (auto& [ch, freq] : freqMap) {
        pq.push(new HuffNode(ch, freq));
    }

    // Edge case: single character
    if (pq.size() == 1) {
        HuffNode* only = pq.top(); pq.pop();
        HuffNode* root = new HuffNode('\0', only->freq);
        root->left = only;
        return root;
    }

    // Step 2: Repeatedly extract two minimum‑frequency nodes, merge them
    while (pq.size() > 1) {
        HuffNode* left = pq.top(); pq.pop();   // smallest
        HuffNode* right = pq.top(); pq.pop();  // second smallest

        // Create internal node with combined frequency
        HuffNode* merged = new HuffNode('\0', left->freq + right->freq);
        merged->left = left;
        merged->right = right;

        pq.push(merged); // push merged node back into heap
    }

    return pq.top(); // root of Huffman tree
}

// Traverse the tree to extract codes (DFS)
void generateCodes(HuffNode* root, const string& code,
                   unordered_map<char, string>& huffCodes) {
    if (!root) return;

    // Leaf node – store the code
    if (!root->left && !root->right) {
        huffCodes[root->ch] = code.empty() ? "0" : code;
        return;
    }

    generateCodes(root->left,  code + "0", huffCodes);
    generateCodes(root->right, code + "1", huffCodes);
}

// Encode a string using Huffman codes
string encode(const string& text, const unordered_map<char, string>& huffCodes) {
    string encoded;
    for (char c : text)
        encoded += huffCodes.at(c);
    return encoded;
}

// Decode a bit string using the Huffman tree
string decode(const string& bits, HuffNode* root) {
    string decoded;
    HuffNode* curr = root;
    for (char b : bits) {
        curr = (b == '0') ? curr->left : curr->right;
        if (!curr->left && !curr->right) { // leaf
            decoded += curr->ch;
            curr = root; // restart from root
        }
    }
    return decoded;
}
```

#### Array‑based version (no explicit tree – code length computation only)

```cpp
#include <vector>
#include <queue>
#include <algorithm>
#include <unordered_map>

using namespace std;

// Returns the weighted path length (total encoded bits)
// without building an explicit tree – useful for quick calculations
int huffmanCost(vector<int> freqs) {
    priority_queue<int, vector<int>, greater<int>> pq(freqs.begin(), freqs.end());

    int totalCost = 0;
    while (pq.size() > 1) {
        int a = pq.top(); pq.pop();
        int b = pq.top(); pq.pop();
        int merged = a + b;
        totalCost += merged; // each merge adds merged frequency to total cost
        pq.push(merged);
    }
    return totalCost;
}
```

### Dry Run Example

**Input characters and frequencies**:

| Character | Frequency |
|-----------|-----------|
| a         | 5         |
| b         | 9         |
| c         | 12        |
| d         | 13        |
| e         | 16        |
| f         | 45        |

**Step‑by‑step construction** (min‑heap contents shown after each step):

| Step | Extract 1 | Extract 2 | Merged Node (freq) | Heap contents after push |
|------|-----------|-----------|---------------------|--------------------------|
| 1    | a (5)     | b (9)     | ab (14)             | {c:12, d:13, ab:14, e:16, f:45} |
| 2    | c (12)    | d (13)    | cd (25)             | {ab:14, e:16, cd:25, f:45} |
| 3    | ab (14)   | e (16)    | abe (30)            | {cd:25, abe:30, f:45} |
| 4    | cd (25)   | abe (30)  | cdabe (55)          | {f:45, cdabe:55} |
| 5    | f (45)    | cdabe (55)| root (100)          | {root:100} |

**Resulting Huffman Tree**:

```
        (100)
       /     \
     f(45)  (55)
           /    \
        (25)    (30)
        / \     / \
     c(12) d(13) (14) e(16)
                 / \
              a(5) b(9)
```

**Generated codes** (left = 0, right = 1):

| Character | Frequency | Huffman Code | Code Length |
|-----------|-----------|--------------|-------------|
| f         | 45        | 0            | 1           |
| c         | 12        | 100          | 3           |
| d         | 13        | 101          | 3           |
| a         | 5         | 1100         | 4           |
| b         | 9         | 1101         | 4           |
| e         | 16        | 111          | 3           |

**Weighted path length** = $5 \times 4 + 9 \times 4 + 12 \times 3 + 13 \times 3 + 16 \times 3 + 45 \times 1 = 20 + 36 + 36 + 39 + 48 + 45 = 224$ bits.

**Fixed‑length encoding** would require $\lceil \log_2 6 \rceil = 3$ bits per character → $100 \times 3 = 300$ bits.

**Compression ratio**: $224 / 300 \approx 74.7\%$ (saves ~25% of bits).

### Core Concepts

- **Prefix‑free code**: No codeword is a prefix of another → uniquely decodable without delimiters. A Huffman code is always prefix‑free by construction (codes are at the leaves of a binary tree).
- **Greedy choice**: At each step, merge the two nodes with the **smallest frequencies**. This ensures that the least frequent characters get the longest codes.
- **Optimal substructure**: If we remove the two least‑frequent symbols and replace them by a single merged symbol, the optimal code for the reduced alphabet extends to an optimal code for the original.
- **Binary tree representation**: Each character is a leaf. Left edge → 0, right edge → 1. The code for a character is the root‑to‑leaf path.
- **Weighted external path length (WEPL)**: The quantity minimised is $\sum_{i} f_i \cdot l_i$ where $f_i$ = frequency and $l_i$ = code length.
- **Variable‑length encoding**: More frequent characters get shorter codes; less frequent ones get longer codes.
- **Full binary tree**: The Huffman tree is always a full binary tree (every internal node has exactly 2 children). If there are $n$ characters, the tree has $n$ leaves and $n-1$ internal nodes.

### Time & Space Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Building frequency map | $O(m)$ | $O(n)$ |
| Building Huffman tree | $O(n \log n)$ | $O(n)$ |
| Generating codes (DFS) | $O(n)$ | $O(n)$ (recursion + codes) |
| Encoding a message | $O(m)$ | $O(m)$ (encoded string) |
| Decoding a message | $O(k)$ | $O(k)$ (decoded string) |

Where $n$ = number of distinct characters, $m$ = length of the original message, $k$ = length of the encoded bit string.

### Detailed Complexity Analysis

- **Building the heap**: Inserting $n$ elements into a min‑heap costs $O(n \log n)$. Alternatively, `heapify` on all $n$ elements is $O(n)$, but since we do $n-1$ extract‑min and insert operations, the overall tree construction is $O(n \log n)$.
- **Merge loop**: Runs $n-1$ iterations. Each iteration does two `extractMin` ($O(\log n)$ each) and one `insert` ($O(\log n)$). Total: $O(n \log n)$.
- **Code generation**: A single DFS traversal of the tree with $2n-1$ nodes → $O(n)$.
- **Encoding**: For each of $m$ characters, look up its code in a hash map ($O(1)$ average) and append. The total encoded length can be at most $O(m \cdot n)$ in the worst case (all codes are $n-1$ bits), but practically much smaller.
- **Decoding**: Walk the tree from root to leaf for each bit. Each bit moves one level → total $O(k)$ where $k$ is the number of encoded bits.
- **Space**: The tree has $2n - 1$ nodes. The code table stores $n$ entries, each code at most $n-1$ bits long. Total space: $O(n^2)$ in the worst case for storing codes, but typically $O(n \log n)$.

### Prerequisites & Properties

- **Input**: A set of characters with known frequencies (or a message from which frequencies are computed).
- **Frequencies must be positive** – zero‑frequency characters are excluded (they need no code).
- **At least 2 distinct characters** are needed for a meaningful Huffman tree (with 1 character, the code is trivially "0").
- **The output is a prefix‑free (instantaneous) code** – the decoder can identify each codeword without look‑ahead.
- **Optimality**: Among all prefix‑free binary codes for a given frequency distribution, Huffman coding produces one with the minimum weighted path length.
- **Not unique**: Multiple valid Huffman trees may exist for the same frequencies (e.g., swapping left/right children, or ties in frequency).
- **Lossless compression**: The original message can be perfectly reconstructed from the encoded bits + the tree (or code table).

### Common Pitfalls & Implementation Traps

- **Forgetting the single‑character case**: If only one distinct character exists, the tree has one leaf. Must assign at least code "0" (a code of length 0 is meaningless).
- **Not using a min‑heap**: Using a max‑heap or unsorted list causes incorrect tree construction or $O(n^2)$ complexity.
- **Memory leaks**: Allocating `HuffNode` with `new` without ever freeing → implement a destructor or use `unique_ptr`.
- **Comparing pointers instead of frequencies** in the priority queue – must provide a custom comparator.
- **Assuming fixed code assignment**: Left = 0 and right = 1 is a convention; swapping is equally valid but must be consistent for encode/decode.
- **Storing codes as integers instead of strings**: Loses leading zeros. E.g., code "010" stored as int 10 (binary) loses the leading 0. Always store as `string`.
- **Transmitting the tree**: The decoder needs the Huffman tree or code table. Forgetting to serialize/transmit the tree means the encoded data is useless.
- **Tie‑breaking**: When two nodes have the same frequency, the merge order is arbitrary but affects the tree shape. Different tie‑breaking rules produce different (equally optimal) codes.
- **Integer overflow**: If frequencies are very large, the merged frequency can overflow `int`. Use `long long` for safety.

### Variations

| Variation | Description | Key Modification |
|-----------|-------------|------------------|
| **Adaptive (Dynamic) Huffman** | Updates the tree as symbols are read, no need for a priori frequency table | Uses FGK or Vitter algorithm; tree is restructured on each symbol |
| **Canonical Huffman Code** | Standardised code assignment based on code lengths | Same tree; codes reassigned so that for same length, codes are consecutive integers |
| **Extended Huffman Coding** | Groups of $k$ symbols encoded together | Alphabet size becomes $|\Sigma|^k$; improves compression but exponential table |
| **n‑ary Huffman Coding** | Uses an $n$‑ary tree instead of binary | Extract $n$ minimum nodes per step; may need dummy nodes if $(|\Sigma|-1) \mod (n-1) \ne 0$ |
| **Length‑Limited Huffman** | Maximum code length bounded by $L$ | Uses Package‑Merge algorithm; NP‑hard in general but polynomial for specific formulations |
| **Huffman with unequal letter costs** | Bit '0' and '1' have different transmission costs | Modified greedy; problem is NP‑hard in general |

### Comparison with Other Algorithms

| Algorithm | Compression Type | Optimality | Time | Key Feature |
|-----------|-----------------|------------|------|-------------|
| **Huffman Coding** | Prefix‑free, symbol‑by‑symbol | Optimal among prefix‑free codes | $O(n \log n)$ | Simple, fast, widely used |
| **Shannon‑Fano Coding** | Prefix‑free, top‑down split | Suboptimal in general | $O(n \log n)$ | Simpler to understand but not optimal |
| **Arithmetic Coding** | Stream‑based, not prefix‑free | Approaches entropy | $O(m)$ | Better compression; encodes entire message as one number |
| **Lempel‑Ziv (LZ77/LZ78)** | Dictionary‑based | Asymptotically optimal | $O(m)$ | No frequency table needed; used in gzip, zip |
| **Run‑Length Encoding (RLE)** | Simple repetition | Optimal for runs | $O(m)$ | Only effective for data with long runs |
| **ANS (Asymmetric Numeral Systems)** | Entropy coding | Near‑entropy | $O(m)$ | Used in zstd, LZFSE; faster than arithmetic coding |

### Theoretical Insights

- **Optimality proof (by induction on $n$, the alphabet size)**:
  - **Base case**: $n = 2$ → assign '0' and '1'. Clearly optimal.
  - **Inductive step**: Let $x, y$ be the two least‑frequent characters. In any optimal tree, $x$ and $y$ can be made siblings at the deepest level (exchange argument). Merging $x, y$ into $z$ with $f_z = f_x + f_y$ reduces the problem to $n-1$ characters. By induction, the Huffman algorithm produces an optimal tree for $n-1$ characters. Expanding $z$ back to $x, y$ as children gives an optimal tree for $n$ characters.

- **Entropy bound**: For a source with entropy $H = -\sum p_i \log_2 p_i$, the average Huffman code length $L$ satisfies:
  $$H \le L < H + 1$$
  This means Huffman coding is within 1 bit per symbol of the theoretical minimum.

- **Shannon's Source Coding Theorem**: No lossless compression scheme can achieve average code length below $H$. Huffman achieves the minimum among integer‑length prefix codes.

- **Kraft's Inequality**: A prefix‑free code with codeword lengths $l_1, l_2, \dots, l_n$ exists if and only if $\sum_{i=1}^n 2^{-l_i} \le 1$. Huffman codes satisfy this with equality when all code lengths are distinct powers.

- **Relationship to optimal binary search trees**: Both problems involve constructing optimal trees; Huffman is greedy ($O(n \log n)$), while optimal BST requires DP ($O(n^3)$).

- **Redundancy**: The redundancy of Huffman coding is $L - H$. It is bounded above by $p_{\max} + \log_2(\frac{2\ln 2}{e}) + \frac{1}{n}$ where $p_{\max}$ is the probability of the most frequent symbol.

### Extended Dry Run – Encoding and Decoding a Message

Using the codes from the previous dry run:

**Encode** the string `"abcdef"`:

| Char | Code  |
|------|-------|
| a    | 1100  |
| b    | 1101  |
| c    | 100   |
| d    | 101   |
| e    | 111   |
| f    | 0     |

Encoded: `1100 1101 100 101 111 0` → `110011011001011110` (18 bits)

Fixed‑length: $6 \times 3 = 18$ bits (same here because each character appears once – compression shines when high‑frequency characters dominate).

**Decode** `110011011001011110` using the tree:

| Bits consumed | Current path | Action | Decoded so far |
|---------------|-------------|--------|----------------|
| 1             | right       | continue | |
| 1             | right→right | continue | |
| 0             | right→right→left | continue | |
| 0             | right→right→left→left | **leaf 'a'** → reset | a |
| 1             | right       | continue | a |
| 1             | right→right | continue | a |
| 0             | right→right→left | continue | a |
| 1             | right→right→left→right | **leaf 'b'** → reset | ab |
| 1             | right       | continue | ab |
| 0             | right→left  | continue | ab |
| 0             | right→left→left | **leaf 'c'** → reset | abc |
| 1             | right       | continue | abc |
| 0             | right→left  | continue | abc |
| 1             | right→left→right | **leaf 'd'** → reset | abcd |
| 1             | right       | continue | abcd |
| 1             | right→right | continue | abcd |
| 1             | right→right→right | **leaf 'e'** → reset | abcde |
| 0             | left        | **leaf 'f'** → reset | abcdef |

**Result**: `abcdef` ✓

### Common Questions

- **Why must the Huffman tree be a full binary tree?**
  If an internal node had only one child, we could remove that node and shorten all codes passing through it, reducing total cost. Hence the optimal tree must be full.

- **Is Huffman coding always better than fixed‑length encoding?**
  Not necessarily – if all characters are equally frequent, Huffman produces codes of nearly equal length, matching fixed‑length. The savings come from skewed frequency distributions.

- **Can Huffman coding produce codes longer than $\lceil \log_2 n \rceil$?**
  Yes. For very skewed distributions (e.g., Fibonacci‑like frequencies), the deepest code can be up to $n-1$ bits for $n$ symbols.

- **How do we handle the tree for the decoder?**
  The tree (or a canonical code table) must be transmitted alongside the encoded data. Canonical Huffman codes minimise the overhead.

- **Why not use Huffman coding everywhere?**
  Huffman is symbol‑by‑symbol and limited by integer code lengths. Arithmetic coding can approach entropy more closely. Also, Huffman requires two passes (one for frequencies, one for encoding).

- **What happens if frequencies change mid‑stream?**
  Static Huffman fails. Use Adaptive (Dynamic) Huffman coding, which updates the tree incrementally.

- **How does Huffman relate to entropy?**
  Huffman's average code length is at most $H + 1$ bits per symbol, where $H$ is the Shannon entropy.

### Edge Cases in Implementation

- **Single character repeated**: e.g., "aaaa" → only one distinct character. Tree has one leaf. Code must be "0" (not empty string). Encoded string = "0000".
- **All characters have equal frequency**: Tree is balanced; all codes have length $\lceil \log_2 n \rceil$ or $\lfloor \log_2 n \rfloor$. No compression benefit.
- **Two characters**: One gets code "0", the other "1". Tree is minimal.
- **Very large alphabet** ($n > 10^5$): Heap operations still efficient. Code table may consume significant memory.
- **Frequency = 0**: Characters with zero frequency should not be included in the tree.
- **Empty input**: No characters → no tree, no encoding. Return empty string.
- **Frequencies that cause integer overflow when summed**: Use `long long` for the merged frequency.
- **Extremely skewed frequencies** (e.g., one character has frequency $2^{30}$, others have 1): Produces a highly unbalanced tree with very long codes for rare characters.

### Useful Code Extensions

```cpp
// 1. Print all Huffman codes
void printCodes(HuffNode* root, const string& code) {
    if (!root) return;
    if (!root->left && !root->right) {
        cout << root->ch << " : " << (code.empty() ? "0" : code) << endl;
        return;
    }
    printCodes(root->left, code + "0");
    printCodes(root->right, code + "1");
}

// 2. Compute average code length
double averageCodeLength(const unordered_map<char, int>& freqMap,
                         const unordered_map<char, string>& huffCodes) {
    int totalFreq = 0;
    double weightedSum = 0.0;
    for (auto& [ch, freq] : freqMap) {
        totalFreq += freq;
        weightedSum += freq * huffCodes.at(ch).size();
    }
    return weightedSum / totalFreq;
}

// 3. Serialize Huffman tree (for transmission with encoded data)
//    Format: '1' + char for leaf, '0' for internal node (pre‑order)
void serialize(HuffNode* root, string& out) {
    if (!root) return;
    if (!root->left && !root->right) {
        out += '1';
        out += root->ch;
    } else {
        out += '0';
        serialize(root->left, out);
        serialize(root->right, out);
    }
}

// 4. Deserialize Huffman tree
HuffNode* deserialize(const string& data, int& idx) {
    if (idx >= (int)data.size()) return nullptr;
    if (data[idx] == '1') {
        idx++;
        return new HuffNode(data[idx++], 0);
    }
    idx++; // skip '0'
    HuffNode* node = new HuffNode('\0', 0);
    node->left = deserialize(data, idx);
    node->right = deserialize(data, idx);
    return node;
}

// 5. Compute compression ratio
double compressionRatio(int originalBits, int encodedBits) {
    return (1.0 - (double)encodedBits / originalBits) * 100.0;
}

// 6. Free Huffman tree memory
void freeTree(HuffNode* root) {
    if (!root) return;
    freeTree(root->left);
    freeTree(root->right);
    delete root;
}
```

### Summary for Revision

- **Problem**: Given characters with frequencies, construct a binary prefix‑free code that minimises the total number of encoded bits ($\sum f_i \cdot l_i$).
- **Greedy strategy**: Repeatedly merge the two lowest‑frequency nodes using a min‑heap.
- **Complexity**: $O(n \log n)$ for tree construction; $O(m)$ for encoding/decoding.
- **Optimality**: Proven via exchange argument + induction. The average code length satisfies $H \le L < H + 1$.
- **Tree structure**: Full binary tree with $n$ leaves (characters) and $n-1$ internal nodes.
- **Prefix‑free property**: Ensures unique decodability – no code is a prefix of another.
- **Key formula**: Weighted path length = $\sum f_i \cdot l_i$ = sum of all internal node frequencies (equivalently, the sum of merged frequencies during construction).
- **Edge cases**: Single character → code "0"; equal frequencies → balanced tree; skewed → deep tree.
- **Compression benefit**: Greatest when frequency distribution is highly skewed.
- **Limitations**: Symbol‑by‑symbol coding; bounded by $H + 1$. Arithmetic coding can do better.
- **Practical use**: JPEG, MP3, DEFLATE (zip/gzip), PKZIP – often combined with LZ77.
- **For decoder**: Must transmit the tree or code table alongside encoded data. Canonical Huffman codes reduce this overhead.
