---
title: How to write template-like data structure in pure C
date: 2013-06-19T20:53:22+00:00
author: "Taras Kushnir"
categories:
  - Programming
keywords:
  - c++
  - data structure
  - generic
  - pure
  - template
aliases:
  - /2013/write-template-like-data-structures-in-pure-c
---
_Problem_:

write type-independent data structure (say, red-black tree, dynamic array etc.) in pure C (which means you cannot use templates and any other features)

_Solution_:

First of all, lets create an interface of our "_generic_" (or "_template_") type. We can interpret `void*` as this unknown type and build some interface functions. It definitely has to have a `constructor` and `destructor`. Also, it can have a `comparator` and kind of `print` function.

So, we can define it like this:

```
typedef void* (*ConstructorFunc)(const void*);
typedef void (*DestructorFunc)(void*);
typedef int (*CompareFunc)(const void*, const void*);
typedef void (*PrintFunc)(const void*);

struct FuncFactory_struct
{
   ConstructorFunc constructor;
   CompareFunc compareFunc;
   DestructorFunc destructorFunc;
   PrintFunc printFunc;
};

#define FuncFactory struct FuncFactory_struct
``` 

And about implementation details..

<!--more-->

Lets assume we have to implement this for some integral type (like `int`) and more complicated one (say, `string`). So, we have to define them and then build our `Factory` structure and fill with these functions:

    void* StringConstructor(const void*);
    /*
    < 0 === [line1 < line2] 
    = 0 === [line1 == line2] 
    > 0 === [line1 > line2]
    */
    int StringComparer(const void*, const void*);
    void StringDestructor(void*);
    void StringPrinter(const void*);
    
    void* IntegerConstructor(const void*);
    int IntegerComparer(const void*, const void*);
    void IntegerDestructor(void*);
    void IntegerPrinter(const void*);

We can now write implementation of this declarations. Each implementation will use `malloc` and `free` for `void*` memory management and then cast it to appropriate type:

    void* IntegerConstructor(const void* from)
    {
       const int* data = (const int*)from;
    
       int* a = (int*)malloc(sizeof(int));
    
       (*a) = (*data);
    
       return a;
    }
    
    int IntegerComparer(const void* op1, const void* op2)
    {
       const int* a = (const int*) op1;
       const int* b = (const int*) op2;
    
       return (*a) - (*b);
    }
    
    void IntegerDestructor(void* a)
    {
       free(a);
       a = 0;
    }
    
    void IntegerPrinter(const void* data)
    {
       const int* a = (const int*)data;
       printf("%d", (*a));
    }

And for String:

    void* StringConstructor(const void* data)
    {
       const char* from = (const char*) data;
       char* str = 0;
    
       int n = strlen(from) + 1;
       str = (char*)malloc(n*sizeof(char));
       return strcpy(str, from);
    }
    
    int StringComparer(const void* line1, const void* line2)
    {
       const char* str1 = (const char*) line1;
       const char* str2 = (const char*) line2;
    
       return strcmp(str1, str2);
    }
    
    void StringDestructor(void* data)
    {
       free(data);
       data = 0;
    }
    
    void StringPrinter(const void* line)
    {
       const char* str = (const char*)line;
       printf(str);
    }

And I'll provide some example of usage. Lets assume we have some tree structure and we have a task to implement  search of a value. Before usage we have to initialize our functions factory. We can just use function names as function pointers.

    void InitIntegerFactoryStruct(FuncFactory* factory)
    {
       InitFactoryStruct(factory,
          IntegerConstructor,
          IntegerDestructor,
          IntegerComparer,
          IntegerPrinter);
    }
    // ....
    FuncFactory integerFunctions;
    AVL_tree* integerTree = 0;
    InitIntegerFactoryStruct(&integerFunctions);
    integerTree = CreateAVLTree(&integerFunctions);

And usage:

    struct AVL_tree_struct
    {
       TreeNode* root;
       TreeNode* zero_node;
       FuncFactory* functions;
    };
    #define AVL_tree struct AVL_tree_struct
    
    TreeNode* FindNode(const AVL_tree* tree, const void* value)
    {
       TreeNode* tn = tree->root;
       CompareFunc cmp = tree->functions->compareFunc;
       while ( (tn != 0) && ( cmp(tn->value, value) != 0 ))
       {
          if (cmp(tn->value, value) > 0)
             tn = tn->left_child;
          else
             tn = tn->right_child;
       }
    
       if (tn == 0)
          return (tree->zero_node);
       else
          if (IsALeftChild(tn) || IsARightChild(tn))
          {
             return tn;
          }
          else if (tn == tree->root)
                return (tree->root);
    
          return (tree->zero_node);
    }
