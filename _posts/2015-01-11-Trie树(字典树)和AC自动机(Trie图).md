---
layout: post
title: Trie树(字典树)和AC自动机(Trie图)
tags: [about, Jekyll, theme, responsive]
comments: true
image:
  feature: 1.jpg
---

今天看了下Trie树、Trie图和AC自动机，它们是用于统计大量字符串问题十分有力的工具。其中AC自动机是在Trie树的基础上建立完成，而Trie图是一种确定性的AC自动机，本质是一个东西。Trie树基本原理是利用字符串的公共前缀来降低查询时间的开销以达到提高效率的目的。这里用两道题来分别说明Trie树和Trie图(AC自动机)具体是怎么应用的。


####题目一：Trie树


在N个单词的词典中，统计某个单词出现的次数（包括前缀）。

例如某词典：bab , babbb , aac , adaac , baby。统计bab出现的次数，输出答案：3


{% highlight c %}

#include <iostream>
using namespace std;

/*******************************************************
Trie树节点
假设所有字符都是a-z之间的26个字符。count用来计数。
根据具体的问题合理设置Trie节点的域。
********************************************************/
struct TrieNode{
	int count;
	TrieNode *next[26] = {NULL};
	TrieNode():count(0){}
};

/*******************************************************
功能：插入一个单词，所有的单词都插入之后，Trie树建立完成
参数：TrieNode *root,string s
	root：Trie树的根
	s   : 待插入的单词
********************************************************/
void insertnode(TrieNode *root,string s){
	int len = s.length();
	TrieNode *curr = root;
	for(int i = 0;i<len;i++){
		int index = s[i] - 'a';
		if(curr->next[index] == NULL){//如果为NULL则需要创建新节点
			curr->next[index] = new TrieNode();
		}
		curr = curr->next[index];
		curr->count++;
	}
}

/*******************************************************
功能：单词查询
参数：TrieNode *root,string s
	root：Trie树的根
	s   ：待查询的单词
********************************************************/
int queryword(TrieNode *root,string s){
	int len = s.length();
	TrieNode *curr = root;
	for(int i = 0;i<len;i++){
		int index = s[i] - 'a';
		if(curr->next[index] == NULL){
			return 0;
		}
		curr = curr->next[index];
	}
	return curr->count;
}


int main(int argc, char **argv)
{
	int N,M;
	TrieNode *root = new TrieNode();
	cin.sync_with_stdio(false);
	cin >> N;   //输入词典的总数
	for(int i = 0;i<N;i++){
		string word;
		cin >> word;
		insertnode(root,word);
	}
	cin >> M;   //输入查询的次数
	for(int i = 0;i<N;i++){
		string word;
		cin >> word;
		cout<<queryword(root,word)<<"\n";
	}
	return 0;
}


{% endhighlight %}


####题目二：Trie图或AC自动机


给定有一个N个单词的敏感词汇表，和一篇文章，判断这篇文章中包含不包含敏感词汇。

例如某词汇表：bab，sb，toobee，ca。和一篇文章askdbasbbsaac，输出答案：true

{% highlight c %}

#include <iostream>
#include <queue>
using namespace std;

/*******************************************************
Trie树节点
假设所有字符都是a-z之间的26个字符。
state表示从根节点到当前节点是否构成一个单词
trie表示匹配失败后的后缀节点。
********************************************************/
struct TrieNode{
	bool state;//为1表示是一个单词
	TrieNode *trie;
	TrieNode *next[26] = {NULL};
	TrieNode():state(false){}
};


/*******************************************************
功能：插入一个单词，所有的单词都插入之后，Trie树建立完成
参数：TrieNode *root,string s
	root：Trie树的根
	s   : 待插入的单词
********************************************************/
void insertnode(TrieNode *root,string s){
	int len = s.length();
	TrieNode *curr = root;
	for(int i = 0;i<len;i++){
		int index = s[i] - 'a';
		if(curr->next[index] == NULL){
			curr->next[index] = new TrieNode();
		}
		curr = curr->next[index];
	}
	curr->state = true;//单词的末尾state为true

}


/*******************************************************
功能：在trie树的基础上建立trie图，基本思想是BFS树的每一个
节点，利用后缀节点trie的next数组来建立当前节点的next数组
类似KMP算法。具体看代码中的注释。
参数：TrieNode *root
	root：Trie树的根
********************************************************/
void TrieGraph(TrieNode *root){
	queue<TrieNode *> q; //BFS
	root->trie = root;  //根节点的失败节点还是根
	for(int i = 0;i<26;i++){
		if(root->next[i] == NULL)
			root->next[i] = root;//没有出现的边指向根
		else{
			//初始化根节点，字符从根节点开始匹配，如果下一层节点失配只能返回到根节点重新匹配。
			root->next[i]->trie = root;
			q.push(root->next[i]);
		}
	}
	while(!q.empty()){  //bfs
		TrieNode *tmp = q.front();
		q.pop();
		for(int i = 0;i<26;i++){
			if(tmp->next[i] == NULL){  
				tmp->next[i] = tmp->trie->next[i];//补全失配时应该转向的节点。
			}
			else{
				//每个节点的trie节点等于其父节点的trie通过边i指向的节点
				//所以节点tmp->next[i]的trie节点应为其父tmp节点的trie的next[i]
				tmp->next[i]->trie = tmp->trie->next[i];
				q.push(tmp->next[i]);
			}
		}
	}
}

/*******************************************************
功能：查询文本s中是否有敏感词汇
参数：TrieNode *root,string s
	root：Trie树的根
	s   ：待查询的文本
********************************************************/
bool querytext(TrieNode *root,string s){
	int len = s.length();
	TrieNode *curr = root;
	for(int i = 0;i<len;i++){
		int index = s[i] - 'a';
		if(curr->state == true) //表示有和谐的词语
			return true;
		curr = curr->next[index];
	}
	return curr->state;//查询到末尾
}

// 测试程序
int main(int argc, char **argv)
{
	int N;
	TrieNode *root = new TrieNode();
	cin.sync_with_stdio(false);
	cin >> N;
	for(int i = 0;i<N;i++){
		string word;
		cin >> word;
		insertnode(root,word);
	}
	TrieGraph(root);
	string str;
	cin >> str;
	cout<<querytext(root,str)<<endl;
	return 0;
}

{% endhighlight %}

<section id="table-of-contents" class="toc">
  <header>
    <h3>Overview</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->
