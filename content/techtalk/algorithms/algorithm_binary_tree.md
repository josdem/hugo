+++
date = "2016-05-23T14:07:56-05:00"
draft = true
title = "Binary Tree"

+++

In computer science, a binary tree is a tree data structure in which each node has at most two children, which are referred to as the left child and the right child.
In this example we will create a rooted binary tree witch has a root node and every node has at most two children, if the node has only one child it is a leaf.


```groovy
interface Node {
  Boolean isRoot()
  Boolean isLeaf()
  List<Node> getChildren()
  void setParent(Node node)
  void addChild(Node node)
}
```

The inteface helps to define our methods in terms of functionality description

```groovy
class NodeImpl implements Node {
  Node parent
  def children = []

  Boolean isRoot(){
    parent == null
  }

  Boolean isLeaf(){
    false
  }

  List<Node> getChildren(){
    children
  }

  void addChild(Node node){
    if(children.size() > 1){
      throw new RuntimeException()
    }
    children.add(node)
  }

  void setParent(Node node){
    this.parent = node
    parent.addChild(this)
  }
}
```

Then we assert our functionality

```groovy
Node root = new NodeImpl()
assert root.isRoot()

Node leftChild = new NodeImpl()
leftChild.setParent(root)
assert !leftChild.isRoot()
assert root.getChildren().size() == 1

Node rightChild = new NodeImpl()
rightChild.setParent(root)
assert !rightChild.isRoot()
assert root.getChildren().size() == 2

Node otherChild = new NodeImpl()
shouldFail(RuntimeException){
  otherChild.setParent(root)
}
```


[Return to the main article](/techtalk/algorithms)
