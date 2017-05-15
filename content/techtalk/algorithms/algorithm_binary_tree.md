+++
date = "2016-05-23T14:07:56-05:00"
draft = true
title = "Binary Tree"

+++

In computer science, a binary tree is a tree data structure in which each node has at most two children, which are referred to as the left child and the right child.
In this example we will create a rooted binary tree witch has a root node and every node has at most two children, if the node has only one child it is a leaf.


```groovy
interface TreeNode {
  Boolean isRoot()
  Boolean isLeaf()
  List<TreeNode> getChildren()
  void setParent(TreeNode node)
  void addChild(TreeNode node)
}
```

The inteface helps to define our methods in terms of functionality description

```groovy
class TreeNodeImpl implements TreeNode {
  TreeNode parent
  def children = []

  Boolean isRoot(){
    parent == null
  }

  Boolean isLeaf(){
    false
  }

  List<TreeNode> getChildren(){
    children
  }

  void addChild(TreeNode node){
    if(children.size() > 1){
      throw new RuntimeException()
    }
    children.add(node)
  }

  void setParent(TreeNode node){
    this.parent = node
    parent.addChild(this)
  }
}
```

Then we assert our functionality

```groovy
import static groovy.test.GroovyAssert.shouldFail

class Launcher {
  
  static void main(String[] args){
    TreeNode root = new TreeNodeImpl()
    assert root.isRoot()

    TreeNode leftChild = new TreeNodeImpl()
    leftChild.setParent(root)
    assert !leftChild.isRoot()
    assert root.getChildren().size() == 1

    TreeNode rightChild = new TreeNodeImpl()
    rightChild.setParent(root)
    assert !rightChild.isRoot()
    assert root.getChildren().size() == 2

    TreeNode otherChild = new TreeNodeImpl()
    shouldFail(RuntimeException){
      otherChild.setParent(root)
    }
  }

}
```

To run the project:

```bash
groovy Launcher.groovy
```

To download the code:

```bash
git clone https://github.com/josdem/algorithms-workshop.git
cd binary-tree
```


[Return to the main article](/techtalk/algorithms)
