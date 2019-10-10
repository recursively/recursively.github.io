---
title: Pointer Manipulations That Should Be Noted
date: 2019-02-12 15:02:09
categories: Programming
tags: C/C++
description: I summarized a few confusing usages of pointers in C and used some code from other guys as examples.
---
## Double pointer
Which means the pointer to pointer. Take a look at this program.
```C
void GetMemory(char *p)
{
   p = (char *)malloc(100);
}
void Test(void)
{
    char *str = NULL;
    GetMemory(str);
    strcpy(str, "hello world");
    printf(str);
}
```
If we directly test this program it will be aborted by throwing an exception. As we all know, the arguments are passed to functions by value in C/C++. So it's clear that the pointer *str* passed into the function *GetMemory* is a copy of real *str*. We can fix this program by using a double pointer.
```C
void GetMemory(char **p)
{
    *p = (char *)malloc(100);
}
void Test(void)
{
    char *str = NULL;
    GetMemory(&str);
    strcpy(str, "hello world\n");
    printf(str);
```

## Simplify code by using pointers
Here gives an implement of linked list:
```C
#include <stdlib.h>
#include <stdio.h>

//节点
typedef struct s_node
{
	//关键字
	int key;
	//下一个节点指针
	struct s_node *next;
} s_node;

//链表
typedef struct s_list
{
	//头节点
	struct s_node *header;
} s_list;

int list_insert(s_list *list, int key)
{
	if (list == NULL)
	{
		return -1;
	}

	//创建新节点
	s_node *n = malloc(sizeof(s_node));
	n->key = key;
	n->next = NULL;

	//如果链表头为空，表头即为新节点
	if (list->header == NULL)
	{
		list->header = n;
		return 0;
	}

	//如果关键字key小于链表头的key
	if (key < list->header->key)
	{
		//替换新节点为链表头，原表头变为新节点的next
		n->next = list->header;
		list->header = n;
		return 0;
	}

	//遍历链表，保持待比较节点的前一个节点
	s_node *p = list->header;
	//找到第一个大于新节点key的节点（注意，此时p其实是这个节点的上一个节点）
	while (p->next != NULL && p->next->key < key)
	{
		p = p->next;
	}
	//插入新节点
	n->next = p->next;
	p->next = n;

	return 0;
}

int main() {
	s_list testlist;
	list_insert(&testlist, 5);
	list_insert(&testlist, 8);
	list_insert(&testlist, 7);
	list_insert(&testlist, 10);
}
```
We must adjust the relations between members of the linked list properly without the help of pointer. This code can be much briefer in the pointer version.
```C
#include <stdlib.h>
#include <stdio.h>

//节点
typedef struct s_node
{
	//关键字
	int key;
	//下一个节点指针
	struct s_node *next;
} s_node;

//链表
typedef struct s_list
{
	//头节点
	struct s_node *header;
} s_list;

int list_insert2(s_list *list, int key)
{
	if (list == NULL)
	{
		return -1;
	}

	//创建新节点
	s_node *n = malloc(sizeof(s_node));
	n->key = key;
	n->next = NULL;

	//二级指针
	s_node **p = &list->header;
	//找到第一个大于key的节点
	while ((*p) != NULL && (*p)->key < key)
	{
		p = &(*p)->next;
	}
	//插入新节点
	n->next = *p;
	*p = n;

	return 0;
}

int main() {
	s_list testlist;
	list_insert2(&testlist, 5);
	list_insert2(&testlist, 8);
	list_insert2(&testlist, 7);
	list_insert2(&testlist, 10);
}
```
It can be clearer by giving a diagram:

<img src="pic_1.png" width="100%" height="100%">

Those examples we saw above is easy to understand. Let's dive in some instances which are a little bit confusing.

## When to pass address?
Take a look at this code:
```C
#include<stdio.h>
#include <string.h>
#include<stdlib.h>
#include <mcheck.h>
#define false 0
#define true 1

//结构体类型，每个导师有三个学生
typedef struct Tea
    {
    char *tName; //导师名字，需要动态分配空间char *====>char
    char **stu;  //三个学生，需要动态分配空间，堆区数组
    int age;
    }Teacher;

//在createTeacher中分配空间
//n1 =3; //导师个数
//n2 = 3 //学生
int createTeacher(Teacher **p, int n1, int n2)
    {
    int len=0;
    int i,j,k;   //分配3个Student信息
    *p = (Teacher*)malloc (sizeof(Teacher)*3);

    for (i=0;i<n1;i++)  //分配3个导师的地址空间
        {
        (*p)[i].tName=(char *)malloc (sizeof(char)*20);  
        if ((*p)[i].tName == NULL)
        {
          return false;
        }
        } 

    //(*p)->stu=(char **)malloc (sizeof(char*)*3);
    for (j=0;j<n2;j++)
        {
         (*p)[j].stu=(char **)malloc (sizeof(char*)*3);
         if ((*p)[j].stu == NULL)
         {
           return false;
         }
         for (k=0;k<3;k++)
         {
         (*p)[j].stu[k]=(char *)malloc (sizeof(char)*20);
         if ((*p)[j].stu[k] == NULL)
             {
             return false;
             }
         }
        }
    return true;
    }

//给成员赋值
void initTeacher(Teacher *p, int n1, int n2)
    {
      int i,j,k=0;  //
      if (p == NULL)
      {
        printf ("error\n");
      }
      //n1 =3; //导师个数
      //n2 = 3 //学生
     puts ("-----导师赋值------");
     for (i=0;i<n1;i++)
     {
        char *buf[]={"王教授","陆教授","田导师"};
        p[i].age=22+i;
        //printf ("%d",p[i].age);
        strcpy (p[i].tName,buf[i]);
    //  printf ("%s",p[i].tName);
        for (j=0;j<n2;j++)
        {

          char *arr[]={"小黎","小田","小张","小王","小胡","小范",
              "小杨","小石","小柯"};
          strcpy (p[i].stu[j],arr[k]);
          ++k;
        //  printf ("%s\n",p[i].stu[j]);
        }
     }
    }

//打印结构体成员信息
void printTeacher(Teacher *p, int n1, int n2)
    {
      int i,j;
      if (p == NULL)
      {
       printf ("error\n");
      }

      for (i=0;i<n1;i++)
      {
        printf ("\t\t%s\n",p[i].tName);  //导师
        for (j=0;j<n2;j++)
        {
           printf ("\t%s",p[i].stu[j]);
        }
        putchar('\n');
        printf ("\t\t%d\n\n",p[i].age);
      }
    }

//释放空间，在函数内部把p赋值为NULL
void freeTeacher(Teacher **p, int n1, int n2)
    {
       int i,j;
       if (p == NULL)
       {
         printf ("Empty\n");
       }
       else
           {
              for (i=0;i<n1;i++)
              {
                 for (j=0;j<n2;j++)
                 {
                   if ((*p)[i].stu[j]!=NULL)
                   {
                     free ((*p)[i].stu[j]);
                     (*p)[i].stu[j]=NULL;
                   }
                 }
                 if ((*p)[i].stu!= NULL)
                 {
                   free ((*p)[i].stu);
                   (*p)[i].stu=NULL;
                 }
                 free ((*p)[i].tName);
                 (*p)[i].tName=NULL;
              }
              free (*p);
              *p=NULL;
           }
    }

int main(void)
    {
    int ret = 0;
    int n1 = 3; //导师个数
    int n2 = 3; //学生
    Teacher *p = NULL;

    setenv("MALLOC_TRACE","1.txt",1);
    mtrace();

    ret = createTeacher(&p, n1, n2);
    if (ret == false)
        {
        printf("createTeacher err:%d\n", ret);
        exit (EXIT_FAILURE);
        }

    initTeacher(p, n1, n2); //给成员赋值
    printTeacher(p, n1, n2);
    //释放空间，在函数内部把p赋值为NULL
    freeTeacher(&p, n1, n2);
    // muntrace();
    // system("pause");
    return 0;
    }
```
I paste the code of function *createTeacher2* below and here comes the question, can we just simply replace the function *createTeacher* to *createTeacher2*?
```C
int createTeacher2(Teacher *p, int n1, int n2)
    {
    int len=0;
    int i,j,k;   //分配3个Student信息
    p = (Teacher*)malloc (sizeof(Teacher)*3);

    for (i=0;i<n1;i++)  //分配3个导师的地址空间
        {
        (p)[i].tName=(char *)malloc (sizeof(char)*20);  
        if ((p)[i].tName == NULL)
        {
          return false;
        }
        } 

    //(*p)->stu=(char **)malloc (sizeof(char*)*3);
    for (j=0;j<n2;j++)
        {
         (p)[j].stu=(char **)malloc (sizeof(char)*3); 
         if ((p)[j].stu == NULL)
         {
           return false;
         }
         for (k=0;k<3;k++)
         {
         (p)[j].stu[k]=(char *)malloc (sizeof(char)*20);
         if ((p)[j].stu[k] == NULL)
             {
             return false;
             }
         }
        }
    return true;
    }
```
Modify the code in *main()*:
```C
ret = createTeacher2(p, n1, n2);
```
This code can be correctly compiled and if we run the program we will get stuck in the step of *initTeacher*. Because the pointer *p* passed to *createTeacher2* wasn't allocated memory due to the pass-by-value feature. In other words, the pointer *p* in the *main()* block will always point to NULL. 

<img src="pic_2.png" width="100%" height="100%">

To fix it, we should use the double pointer to pass the address of pointer *p*.

There is another example we should take care of. Here is an implement of a binary search tree.
```C
//binarysearchtree.h
#ifndef __BINARY_SEARCH_TREE__
#define __BINARY_SEARCH_TREE__
typedef int mytype;

typedef struct _bstree_node
{
	mytype data;
	struct _bstree_node *lchild;
	struct _bstree_node *rchild;
}bstree_node;

typedef struct _bstree
{
    int size;
    int (*compare)(mytype key1,mytype key2);
	int (*destory)(mytype data);
	bstree_node *root;
}bstree;

typedef int (*compare_fuc)(mytype key1,mytype key2);
typedef int (*destory_fuc)(mytype data);

#define bstree_is_empty(tree)  (tree->size == 0)

bstree *bstree_create(compare_fuc compare,destory_fuc destory);

#endif
```
```C
//binarysearchtree.c
#include<assert.h>
#include<string.h>
#include<stdlib.h>
#include<stdio.h>
#include"binarysearchtree.h"

bstree *bstree_create(compare_fuc compare,destory_fuc destory)
{
	bstree *tree = NULL;

	tree = (bstree*)malloc(sizeof(bstree));
	if (tree == NULL)
	{
		return NULL;
	}

	tree->size = 0;
    tree->compare = compare;
    tree->destory = destory;
	tree->root = NULL;
    return tree;
}

bstree_node *bstree_search(bstree *tree,mytype data)
{
	bstree_node *node = NULL;
	int res = 0;

	if ((tree == NULL) || (bstree_is_empty(tree)))
	{
		return NULL;
	}
    node = tree->root;

	while(node != NULL)
	{
		res = tree->compare(data,node->data);
		if(res == 0)
		{
			return node;
		}
		else if (res > 0)
		{
			node = node->rchild;
		}
		else
		{
			node = node->lchild;
		}
	}
   
	return NULL;
}

int bstree_insert(bstree * tree, mytype data)
{
	printf("second ptr address: %p", &tree);
    bstree_node *node = NULL;
    bstree_node *tmp = NULL;
	int res = 0;

	if (tree == NULL)
	{
		return -1;
	}

	node = (bstree_node *)malloc(sizeof(bstree_node));
	if (node == NULL)
	{
		return -2;
	}

	node->data = data;
	node->lchild = NULL;
	node->rchild = NULL;

	/*如果二叉树为空，直接挂到根节点*/
	if (bstree_is_empty(tree))
	{
        tree->root = node;
		tree->size++;
		return 0;
	}

	tmp = tree->root;

	while(tmp != NULL)
	{
		res = tree->compare(data,tmp->data);
		if (res > 0) /*去右孩子查找*/
		{
			if (tmp->rchild == NULL)
			{
				tmp->rchild = node;
				tree->size++;
				return 0;
			}
		    tmp = tmp->rchild;
		}
		else /*去左孩子查找*/
		{
			if(tmp->lchild == NULL)
			{
				tmp->lchild = node;
				tree->size++;
				return 0;
			}
			tmp = tmp->lchild;
		}
	}
    
	return -3;
}

int bstree_delete(bstree *tree,mytype data)
{
	bstree_node *node = NULL;/*要删除的节点*/
	bstree_node *pnode = NULL;/*要删除节点的父节点*/
	bstree_node *minnode = NULL;/*要删除节点的父节点*/
	bstree_node *pminnode = NULL;/*要删除节点的父节点*/
    mytype tmp = 0;
	int res = 0;

	if ((tree == NULL) || (bstree_is_empty(tree)))
	{
		return -1;
	}

	node = tree->root;
	while ((node != NULL) && ((res = tree->compare(data,node->data)) != 0))
	{
		pnode = node;
		if(res > 0)
		{
            node = node->rchild;
		}
		else
		{
            node = node->lchild;
		}
	}
	/*说明要删除的节点不存在*/
	if (node == NULL)
	{
		return -2;
	}

    /*1、如果要删除node有2个子节点，需要找到右子树的最小节点minnode，
	 * 更新minnode和node节点数据，这样minnode节点就是要删除的节点
	 * 再更新node和pnode节点指向要删除的节点*/
	if ((node->lchild != NULL) && (node->rchild != NULL))
	{
		minnode = node->rchild;
		pminnode = node;

		while(minnode->lchild != NULL)
		{
			pminnode = minnode;
			minnode = minnode->lchild;
		}

		/*node 节点和minnode节点数据互换*/
        tmp = node->data;
		node->data = minnode->data;
		minnode->data = tmp;
		/*更新要删除的节点和其父节点*/
		node = minnode;
		pnode = pminnode;
	}

	/*2、当前要删除的节点只有左孩子或者右孩子时，直接父节点的直向删除的节点*/
	if (node->lchild != NULL)
	{
        minnode = node->lchild;
	}
	else if (node->rchild != NULL)
	{
		minnode = node->rchild;
	}
	else
	{
		minnode = NULL;
	}

	if (pnode == NULL)/*当要删除的时根节点时,*/
	{
		tree->root = minnode;
	}
	else if (pnode->lchild == node)
	{
		pnode->lchild = minnode;
	}
	else
	{
		pnode->rchild = minnode;
	}
    tree->size--;
	free (node);

	return 0;
}

/*采用递归方式删除节点*/
void bstree_destory_node(bstree *tree,bstree_node *root)
{
	if (root == NULL)
	{
		return;
	}

	bstree_destory_node(tree,root->lchild);
	bstree_destory_node(tree,root->rchild);
	free(root);
}

/*二叉搜索树销毁*/
void bstree_destory(bstree *tree)
{
	bstree_destory_node(tree,tree->root);
	free(tree);
	return;
}

/*中序遍历打印树节点*/
void bstree_inorder_node(bstree_node *root)
{
	bstree_node *node = NULL;
	if (root == NULL)
	{
		return;
	}

	bstree_inorder_node(root->lchild);
	printf(" %d ",root->data);
	bstree_inorder_node(root->rchild);
    return;
}

void bstree_dump(bstree *tree)
{
	bstree_node *node = NULL;
	if ((tree == NULL) || (bstree_is_empty(tree)))
	{
		printf("\r\n 当前树是空树");
	}
	printf("\r\nSTART-----------------%d------------\r\n",tree->size);
	bstree_inorder_node(tree->root);
	printf("\r\nEND---------------------------------",tree->size);

}

int bstree_compare(mytype key1,mytype key2)
{
	if (key1 == key2)
	{
		return 0;
	}
	else if (key1 > key2)
	{
		return 1;
	}
	else
	{
		return -1;
	}
}

int main()
{
	bstree *tree = NULL;
	bstree_node *node = NULL;
	mytype data = 0;
	int res = 0;

	setenv("MALLOC_TRACE","1.txt",1);
    mtrace();

	tree = bstree_create(bstree_compare,NULL);
	printf("first address: %d", tree);
	assert(tree != NULL);

    while(1)
	{
		printf("\r\n插入一个数字，输入100时退出：");
		scanf("%d",&data);
		if(data == 100)break;
		res = bstree_insert(tree,data);
		printf("\r\n %d 插入%s成功",data,(res != 0)?("不"):(" "));
	}
	bstree_dump(tree);

    while(1)
	{
		printf("\r\n查询一个数字，输入100时退出：");
		scanf("%d",&data);
		if(data == 100)break;
		node = bstree_search(tree,data);
		printf("\r\n %d %s存在树中",data,(node == NULL)?("不"):(" "));
	}
	bstree_dump(tree);
    while(1)
	{
		printf("\r\n删除一个数字，输入100时退出：");
		scanf("%d",&data);
		if(data == 100)break;
		res = bstree_delete(tree,data);
		printf("\r\n %d 删除%s成功",data,(res != 0)?("不"):(" "));
	    bstree_dump(tree);
	}

	bstree_destory(tree);

    muntrace();

	return 0;
}
```
Now let's focus on the first while loop among the *main()* block. In this case, the pointer *tree* was directly passed into the function *bstree_insert*, it's kind of odd compared with the previous instances. The difference is that the pointer *tree* has been allocated memory before passed as an argument. 

<img src="pic_3.png" width="100%" height="100%">

There are actually two pointers which belong to two different address in the memory, but both of them point to the same space. If we change the value of the space one pointer points to, we simultaneously change the value that the other pointer points to. If the extreme performance is what you want, The function *bstree_insert* can be correctly replaced by *bstree_insert2* as below. This conversion can save the time of memory allocation.
```C
int bstree_insert2(bstree ** tree, mytype data)
{
	printf("second address: %d", *tree);
    bstree_node *node = NULL;
    bstree_node *tmp = NULL;
	int res = 0;

	if (tree == NULL)
	{
		return -1;
	}

	node = (bstree_node *)malloc(sizeof(bstree_node));
	if (node == NULL)
	{
		return -2;
	}

	node->data = data;
	node->lchild = NULL;
	node->rchild = NULL;

	/*如果二叉树为空，直接挂到根节点*/
	if (bstree_is_empty((*tree)))
	{
        (*tree)->root = node;
		(*tree)->size++;
		return 0;
	}

	tmp = (*tree)->root;

	while(tmp != NULL)
	{
		res = (*tree)->compare(data,tmp->data);
		if (res > 0) /*去右孩子查找*/
		{
			if (tmp->rchild == NULL)
			{
				tmp->rchild = node;
				(*tree)->size++;
				return 0;
			}
		    tmp = tmp->rchild;
		}
		else /*去左孩子查找*/
		{
			if(tmp->lchild == NULL)
			{
				tmp->lchild = node;
				(*tree)->size++;
				return 0;
			}
			tmp = tmp->lchild;
		}
	}
    
	return -3;
}
```
Then modify the *main()* code:
```C
res = bstree_insert2(&tree,data);
```
One last instance that we should focus on, there is an implement of a binary tree:
```C
//binarytree.c
#include<assert.h>
#include<string.h>
#include<stdlib.h>
#include<stdio.h>
#include"list_queue.h"

typedef struct _treenode
{
	int data;
	struct _treenode *lchild;
	struct _treenode *rchild;
}Tnode,Tree;

void binarytree_create(Tree **Root)
{
	int a = 0;
	printf("\r\n输入节点数值((当输入为100时，当前节点创建完成))):");
	scanf("%d",&a);


	if (a == 100)
	{
		*Root = NULL;
	}
	else
	{
		*Root = (Tnode *)malloc(sizeof(Tnode));
		if (*Root == NULL)
		{
			return;
		}

		(*Root)->data = a;
		printf("\r\n create %d 的左孩子:",a);
		binarytree_create(&((*Root)->lchild));
		printf("\r\n create %d 的右孩子:",a);
		binarytree_create(&((*Root)->rchild));
	}

	return ;
}

void binarytree_destory(Tree *root)
{
	if (root == NULL)
	{
		return;
	}

	binarytree_destory(root->lchild);
	binarytree_destory(root->rchild);
	free(root);
}

/*先序遍历:根结点--》左子树---》右子树*/
void binarytree_preorder(Tree *root)
{
	if (root == NULL)
	{
		return;
	}
	printf(" %d ",root->data);
	binarytree_preorder(root->lchild);
	binarytree_preorder(root->rchild);
    return;
}
/*中序遍历:左子树--》跟节点---》右子树*/
void binarytree_inorder(Tree *root)
{
	if (root == NULL)
	{
		return;
	}
	binarytree_inorder(root->lchild);
	printf(" %d ",root->data);
	binarytree_inorder(root->rchild);
    return;
}
/*后序遍历:左子树---》右子树-》根节点*/
void binarytree_postorder(Tree *root)
{
	if (root == NULL)
	{
		return;
	}
	binarytree_postorder(root->lchild);
	binarytree_postorder(root->rchild);
	printf(" %d ",root->data);
    return;
}

void binarytree_levelorder(Tree * root)
{
	list_queue *queue = NULL;
	Tnode * node = NULL;

	if(root == NULL)
	{
		return;
	}

	queue = list_queue_create();

	/*根节点先入队*/
	list_queue_enqueue(queue,(void *)root);

	while(!list_queue_is_empty(queue))
	{
		list_queue_dequeue(queue,(void *)&node);
		printf(" %d ",node->data);

		if(node->lchild != NULL)
		{
			list_queue_enqueue(queue,(void *)node->lchild);
		}

		if(node->rchild != NULL)
		{
			list_queue_enqueue(queue,(void *)node->rchild);
		}
	}

	free(queue);

}
/*打印叶子节点*/
void binarytree_printfleaf(Tree *root)
{
	if (root == NULL)
	{
		return;
	}

	if ((root->lchild == NULL) && (root->rchild == NULL))
	{
		printf(" %d ",root->data);
	}
	else
	{
		binarytree_printfleaf(root->lchild);
		binarytree_printfleaf(root->rchild);
	}
}
/*打印叶子的个数*/
int binarytree_getleafnum(Tree*root)
{
	if (root == NULL)
	{
		return 0;
	}

	if ((root->lchild == NULL) && (root->rchild == NULL))
	{
		return 1;
	}
	
	return binarytree_getleafnum(root->lchild) + binarytree_getleafnum(root->rchild);

}
/*打印数的高度*/
int binarytree_gethigh(Tree *root)
{
	int lhigh = 0;
	int rhigh = 0;
	
	if (root == NULL)
	{
		return 0;
	}

	lhigh = binarytree_gethigh(root->lchild);
	rhigh = binarytree_gethigh(root->rchild);

	return ((lhigh > rhigh)?(lhigh + 1):(rhigh + 1));
}

int main()
{
	Tree *root = NULL;

	setenv("MALLOC_TRACE","1.txt",1);
    mtrace();
	
	printf("\r\n创建二叉树:");
	binarytree_create(&root);
	printf("\r\n先序遍历二叉树:");
	binarytree_preorder(root);
	printf("\r\n中序遍历二叉树:");
	binarytree_inorder(root);
	printf("\r\n后序遍历二叉树:");
	binarytree_postorder(root);
	printf("\r\n层次遍历二叉树:");
	binarytree_levelorder(root);

	printf("\r\n打印二叉树叶子节点:");
	binarytree_printfleaf(root);
	printf("\r\n打印二叉树叶子节点个数:%d",binarytree_getleafnum(root));
	printf("\r\n打印二叉树高度:%d",binarytree_gethigh(root));

	binarytree_destory(root);

	return 0;
}
```
```C
//list_queue.h
#ifndef LINK_LIST_QUEUE_H
#define LINK_LIST_QUEUE_H

typedef struct _list_queue_node
{
	void *data;
	struct _list_queue_node *next;
}queue_node;

typedef struct _list_queue
{
	int num;
	queue_node *head;
	queue_node *tail;
}list_queue;

#define list_queue_is_empty(queue) ((queue->num) == 0)
list_queue *list_queue_create();
int list_queue_enqueue(list_queue *queue,void *data);
int list_queue_dequeue(list_queue *queue,void **data);

#endif
```
```C
//list_queue.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include"./list_queue.h"

/*创建队列头*/
list_queue *list_queue_create()
{
	list_queue * queue = NULL;

	queue = (list_queue *)malloc(sizeof(list_queue));
	if(queue == NULL)
	{
		return NULL;
	}

	queue->num  = 0;
	queue->head = NULL;
	queue->tail = NULL;

	return queue;
}
int list_queue_enqueue(list_queue *queue,void *data)
{
	queue_node *ptmp = NULL;

	if(queue == NULL)
	{
		return -1;
	}

	ptmp = (queue_node *)malloc(sizeof(queue_node));
	if (ptmp == NULL)
	{
		return -1;
	}

	ptmp->data = data;
	ptmp->next = NULL;
	if (queue->head == NULL)
	{
		queue->head = ptmp;
	}
	else
	{
	    queue->tail->next = ptmp;

	}
	queue->tail = ptmp;
	queue->num++;

	return 0;
}

/*出队*/
int list_queue_dequeue(list_queue *queue,void **data)
{
	queue_node * ptmp = NULL;

	if ((queue == NULL) || (data == NULL) || list_queue_is_empty(queue))
	{
		return -1;
	}

	*data = queue->head->data;
    ptmp = queue->head;
	queue->head = queue->head->next;
	queue->num--;

	if (queue->head == NULL)
	{
		queue->tail = NULL;
	}

	
	free(ptmp);
	return 0;
}
```
Let's review the code of *binarytree.c* and replace the function *binarytree_create* by *binarytree_create2*:
```C
void binarytree_create2(Tree *Root)
{
	int a = 0;
	printf("\r\n输入节点数值((当输入为100时，当前节点创建完成))):");
	scanf("%d",&a);


	if (a == 100)
	{
		Root = NULL;
	}
	else
	{
		Root = (Tnode *)malloc(sizeof(Tnode));
		if (Root == NULL)
		{
			return;
		}

		Root->data = a;
		printf("\r\n create %d 的左孩子:",a);
		binarytree_create(Root->lchild);
		printf("\r\n create %d 的右孩子:",a);
		binarytree_create(Root->rchild);
	}

	return;
}
```
```C
binarytree_create(root);
```
Is that right? definitely not. The code can be surely compiled successfully, but the procedure will stop at function *binarytree_preorder*. Remember? The pointer cannot be directly passed as an argument if there will be memory allocation to the pointer later. the original pointer will always stay NULL. In addition, remember to free memory in each example to avoid memory leaks if I didn't do that.