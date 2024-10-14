---
date: "2024-10-13"
title: "Who's in the Shahnameh? Tries Can Help."
tags: ["go", "tries", "algorithms"]
author: "masoudkf"
description: "Tries are tree-like data structures used in applications such as autocomplete and spell checkers. For example, is there a character in the Shahnameh whose name starts with a Z? Well, let's see."
---

<br/>
<p align="center">
  <img src="https://res.cloudinary.com/mkf/image/upload/v1728863652/shahnameh_mkbmzm.png" width="600"/>
</p>
<br/>

In case you're not familiar with the Shahnameh, it's a book about Persian kings (Shahnameh means The Book of Kings). It's a masterpiece written by the Persian poet Ferdowsi. There are many stories in the book, and of course, so many characters. Read more on [Wikipedia](https://en.wikipedia.org/wiki/Shahnameh).

We want to create an application (in Go, of course) that gives us autocompletion when we're searching for a character name. For instance, when I start typing `ma`, it gives me `Manijeh`, `Manuchehr`, and `Mardas`. How? Tries to the rescue.

## Tries
Tries are tree-like data structures used in applications such as autocomplete and spell checkers. They have a root, which doesn't hold any information, other than it's where you begin traversing the tree, and a number of nodes where each node contains a character from your dictionary. Read more on [Wikipedia](https://en.wikipedia.org/wiki/Trie).

### Node Relations
A parent-child relationship in a trie represents two consecutive characters where the parent comes before the child. For instance, if the parent node has the character `i` and the child node represents a `d`, then together they create `id`.


<br/>
<p align="center">
  <img src="https://res.cloudinary.com/mkf/image/upload/v1728860389/Untitled_Diagram.drawio_osyqmm.png" width="300"/>
</p>
<p style="text-align: center"><i>A Trie</i></p>
<br/>

Look at the trie above. It has 4 different words in it: `masoud`, `master`, `lake`, and `late`. As you can see, words can share prefixes, and that makes tries efficient for prefix lookups. Given a prefix, it can find all the words that share that prefix in linear time `O(N)` where `N` is the length of the longest word found, whereas a data structure like a list, or even a hash map, can solve the problem in `O(N)` where `N` is the size of the dictionary. So, big difference. Also, in terms of memory, tries are more efficient as they don't store the same prefix twice.

Aside from the character that each node holds, it also contains a flag to show if it's the end character or not. For example, in the trie above, characters `d`, `r`, `e`, and `e` (different `e`) are end characters. It's important to have this property on the nodes to distinguish between words and a sequence of characters.

### Child Nodes
As shown in the above figure, each child node holds a character from the dictionary, so it's important to know your characters. Usually, you create a hash map for this.

## Go Implementation
We start by declaring the tree and its nodes:

```go
// trieNode represents a node in a trie.
type trieNode struct {
	children []*trieNode
	// the character this nodes represents.
	char     rune
	// is this the end character of a word.
	isEnd    bool
}

// trie represents a trie data structure.
type trie struct {
	root *trieNode
}

// NewTrie creates a new trie data structure.
func NewTrie() *trie {
	return &trie{
		root: NewTrieNode(-1),
	}
}

// NewTrieNode create a new trie node given a character.
func NewTrieNode(char rune) *trieNode {
	return &trieNode{
		char:     char,
		children: make([]*trieNode, len(alphabetMap)),
	}
}
```

Nothing really special here. Just a tree that has a root, and a bunch of nodes where each node can have a list of children. The last two are initializers to create a new trie and new nodes.

### Dictionary Characters
As mentioned above, you need to know your characters and how many there are. Otherwise, you can't achieve linear lookups. In this step, you need to make sure you can see if a character exists as a child of a parent node in constant time.

```go
alphabetMap = map[rune]int{
	'a': 0,
	'b': 1,
	'c': 2,
	'd': 3,
	'e': 4,
	'f': 5,
	'g': 6,
	'h': 7,
	'i': 8,
	'j': 9,
	'k': 10,
	'l': 11,
	'm': 12,
	'n': 13,
	'o': 14,
	'p': 15,
	'q': 16,
	'r': 17,
	's': 18,
	't': 19,
	'u': 20,
	'v': 21,
	'w': 22,
	'x': 23,
	'y': 24,
	'z': 25,
	// adding space because some of Shahnameh characters
	// have space in them.
	' ': 26,
}
```

### Populating the Trie
Before using a trie, it needs to include your dictionary of words. So, you need an insert method. 

```go
// Insert adds a word to the trie, provided all its
// characters are supported.
func (t *trie) Insert(word string) error {
	word = strings.ToLower(word)
	current := t.root
	for _, char := range word {
		// checking if the character exists in constant time.
		charIndex, ok := alphabetMap[char]
		if !ok {
			return errors.New("character not supported")
		}
		// getting the character index (if exists) in constant time.
		c := current.children[charIndex]
		if c != nil {
			current = c
		} else {
			current.children[charIndex] = NewTrieNode(char)
			current = current.children[charIndex]
		}
	}

	// the current node holds the end character of the word.
	current.isEnd = true

	return nil
}
```

We start from the `root` (almost always), and traverse the tree to see if we need to create a new node for a character. As long as the prefix already exists, we move to the next character by updating `current`. Once we find that this is a new prefix, we create a new trie node.

### Autocomplete
Autocomplete is finding all the words that share the same prefix. So, given a prefix, or query, we want to find those words in the trie.


```go
// Autocomplete returns all the words that share the prefix `query`.
func (t *trie) Autocomplete(query string) []string {
	suggestions := make([]string, 0)
	query = strings.ToLower(query)
	current := t.root

	for _, char := range query {
		charIndex := alphabetMap[char]
		c := current.children[charIndex]
		if c == nil {
			return suggestions
		}
		current = c
	}

	return current.findSuggestions(query)
}

// findSuggestions finds all the words that share the prefix `query`,
// starting from the trieNode it's called on.
func (tn *trieNode) findSuggestions(query string) []string {
	var suggestions []string

	if tn.isEnd {
		suggestions = append(suggestions, query)
	}

	for _, childNode := range tn.children {
		if childNode != nil {
			suggestions = append(suggestions, childNode.findSuggestions(query+string(childNode.char))...)
		}
	}

	return suggestions
}
```

The `Autocomplete` method takes a prefix and first finds the node at the __end__ of the prefix. For example, if the prefix is `la`, we first need to find `l`, and then `a`, and then find all the words that share this prefix. 

Once the node that holds the last character of the prefix is found, the `findSuggestions` method is called on the node. It uses depth-first search (DFS) to find all the words with the same prefix. (recursion anyone?). And that's all there's to it. 

There are other methods you can define on the Trie, such as find, or spellcheck, which is basically the same thing.

## Demo
For the demo, I needed the character names from the Shahnameh to create my dictionary. I found a page on Wikipedia and turned the list into a Go slice. You can find the gist [here](https://gist.github.com/masoudkarimif/1c7ba515aa0c67ba1f82c63f1f9bfa1a).

After that, it was populating the trie and hooking it up to a front-end. See the demo [here](https://shahnameh-names.masoudkf.com/), and read the book too if you get a chance. It's fascinating!
