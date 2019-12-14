---
title: "ARTS (1202-1208)"
date: 2019-12-14T22:54:39+08:00
draft: false
---

## Algorithm

### Problem

Gopl Exercise 5.1

书中写了一个程序，给定一段 HTML 源码，能够提取出其中的 `<a href>` 链接内容。解析 HTML document 是直接的调用库函数，而这个程序做的事就是遍历库函数返回的 document tree, 找出其中的 `<a href>`.

```
func main() {
	doc, err := html.Parse(os.Stdin)
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlinks1: %v\n", err)
		os.Exit(1)
	}
	for _, link := range dfsVisit(nil, doc) {
		fmt.Println(link)
	}
}

// visit appends to links each link found in n and returns the result.
func visit(links []string, n *html.Node) []string {
	if n.Type == html.ElementNode && n.Data == "a" {
		for _, a := range n.Attr {
			if a.Key == "href" {
				links = append(links, a.Val)
			}
		}
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
		links = visit(links, c)
	}
	return links
}
```

Exercise 5.1 要求把以上代码的 `for c := n.FirstChild; c != nil; c = c.NextSibling {}` 改为递归实现。

### Solution

https://github.com/JuneYuan/gopl.io/tree/master/ch5/findlinks1

## Review


## Tip

To Be Done.

Iteration variable caveat in Golang.