---
title: How to write template-like data structure in pure C
date: 2013-06-19T20:53:22+00:00
author: "Taras Kushnir"
permalink: /write-template-like-data-structures-in-pure-c/
tagazine-media:
  - 'a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";i:0;s:6:"author";s:8:"20401582";s:7:"blog_id";s:8:"53632187";s:9:"mod_stamp";s:19:"2013-06-19 19:09:11";}'
categories:
  - Programming
tags:
  - c++
  - data structure
  - generic
  - pure
  - template
---
_Problem_:

write type-independent data structure (say, red-black tree, dynamic array etc.) in pure C (which means you cannot use templates and any other features)

_Solution_:

First of all, lets create an interface of our "_generic_" (or "_template_") type. We can interpret `void*` as this unknown type and build some interface functions. It definitely has to have a `constructor` and `destructor`. Also, it can have a `comparator` and kind of `print` function.

So, we can define it like this:

    <span style="color:#000080;">typedef</span> <span style="color:#008000;">void</span>* (*<span style="color:#008080;">ConstructorFunc</span>)(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*);
    <span style="color:#000080;">typedef</span> <span style="color:#008000;">void</span> (*<span style="color:#008080;">DestructorFunc</span>)(<span style="color:#008000;">void</span>*);
    <span style="color:#000080;">typedef</span> <span style="color:#008000;">int</span> (*<span style="color:#008080;">CompareFunc</span>)(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*, <span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*);
    <span style="color:#000080;">typedef</span> <span style="color:#008000;">void</span> (*<span style="color:#008080;">PrintFunc</span>)(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*);
    
    <span style="color:#000080;">struct</span> <span style="color:#3366ff;">FuncFactory_struct</span>
    {
       <span style="color:#3366ff;">ConstructorFunc</span> constructor;
       <span style="color:#3366ff;">CompareFunc</span> compareFunc;
       <span style="color:#3366ff;">DestructorFunc</span> destructorFunc;
       <span style="color:#3366ff;">PrintFunc</span> printFunc;
    };
    
    <span style="color:#000080;">#define</span> <span style="color:#008080;">FuncFactory</span> <span style="color:#000080;">struct</span> <span style="color:#3366ff;">FuncFactory_struct
    </span>

And about implementation details..

<!--more-->

Lets assume we have to implement this for some integral type (like `int`) and more complicated one (say, `string`). So, we have to define them and then build our `Factory` structure and fill with these functions:

    <span style="color:#008000;">void</span>* StringConstructor(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*);
    /*
    < 0 === [line1 < line2] 
    = 0 === [line1 == line2] 
    > 0 === [line1 > line2]
    */
    <span style="color:#008000;">int</span> StringComparer(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*, <span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*);
    <span style="color:#008000;">void</span> StringDestructor(<span style="color:#008000;">void</span>*);
    <span style="color:#008000;">void</span> StringPrinter(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*);
    
    <span style="color:#008000;">void</span>* IntegerConstructor(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*);
    <span style="color:#008000;">int</span> IntegerComparer(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*, <span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*);
    <span style="color:#008000;">void</span> IntegerDestructor(<span style="color:#008000;">void</span>*);
    <span style="color:#008000;">void</span> IntegerPrinter(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>*);

We can now write implementation of this declarations. Each implementation will use `malloc` and `free` for `void*` memory management and then cast it to appropriate type:

    <span style="color:#008000;">void</span>* IntegerConstructor(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>* from)
    {
       <span style="color:#000080;">const</span> <span style="color:#008000;">int</span>* data = (<span style="color:#000080;">const</span> <span style="color:#008000;">int</span>*)from;
    
       <span style="color:#008000;">int</span>* a = (<span style="color:#008000;">int</span>*)malloc(sizeof(<span style="color:#008000;">int</span>));
    
       (*a) = (*data);
    
       <span style="color:#000080;">return</span> a;
    }
    
    <span style="color:#008000;">int</span> IntegerComparer(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>* op1, <span style="color:#000080;">const</span> <span style="color:#008000;">void</span>* op2)
    {
       <span style="color:#000080;">const</span> <span style="color:#008000;">int</span>* a = (<span style="color:#000080;">const</span> <span style="color:#008000;">int</span>*) op1;
       <span style="color:#000080;">const</span> <span style="color:#008000;">int</span>* b = (<span style="color:#000080;">const</span> <span style="color:#008000;">int</span>*) op2;
    
       return (*a) - (*b);
    }
    
    <span style="color:#008000;">void</span> IntegerDestructor(<span style="color:#008000;">void</span>* a)
    {
       free(a);
       a = 0;
    }
    
    <span style="color:#008000;">void</span> IntegerPrinter(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>* data)
    {
       <span style="color:#000080;">const</span> <span style="color:#008000;">int</span>* a = (<span style="color:#000080;">const</span> <span style="color:#008000;">int</span>*)data;
       printf(<span style="color:#993300;">"%d"</span>, (*a));
    }

And for String:

    <span style="color:#008000;">void</span>* StringConstructor(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>* data)
    {
       <span style="color:#000080;">const</span> <span style="color:#008000;">char</span>* from = (<span style="color:#000080;">const</span> <span style="color:#008000;">char</span>*) data;
       <span style="color:#008000;">char</span>* str = 0;
    
       <span style="color:#008000;">int</span> n = strlen(from) + 1;
       str = (<span style="color:#008000;">char</span>*)malloc(n*sizeof(<span style="color:#008000;">char</span>));
       <span style="color:#000080;">return</span> strcpy(str, from);
    }
    
    <span style="color:#008000;">int</span> StringComparer(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>* line1, <span style="color:#000080;">const</span> <span style="color:#008000;">void</span>* line2)
    {
       <span style="color:#000080;">const</span> <span style="color:#008000;">char</span>* str1 = (<span style="color:#000080;">const</span> <span style="color:#008000;">char</span>*) line1;
       <span style="color:#000080;">const</span> <span style="color:#008000;">char</span>* str2 = (<span style="color:#000080;">const</span> <span style="color:#008000;">char</span>*) line2;
    
       <span style="color:#000080;">return</span> strcmp(str1, str2);
    }
    
    <span style="color:#008000;">void</span> StringDestructor(<span style="color:#008000;">void</span>* data)
    {
       free(data);
       data = 0;
    }
    
    <span style="color:#008000;">void</span> StringPrinter(<span style="color:#000080;">const</span> <span style="color:#008000;">void</span>* line)
    {
       <span style="color:#000080;">const</span> <span style="color:#008000;">char</span>* str = (<span style="color:#000080;">const</span> <span style="color:#008000;">char</span>*)line;
       printf(str);
    }

And I'll provide some example of usage. Lets assume we have some tree structure and we have a task to implement  search of a value. Before usage we have to initialize our functions factory. We can just use function names as function pointers.

    <span style="color:#003366;">void</span> InitIntegerFactoryStruct(<span style="color:#008080;">FuncFactory</span>* factory)
    {
       InitFactoryStruct(factory,
          IntegerConstructor,
          IntegerDestructor,
          IntegerComparer,
          IntegerPrinter);
    }
    // ....
    <span style="color:#008080;">FuncFactory</span> integerFunctions;
    <span style="color:#008080;">AVL_tree</span>* integerTree = 0;
    InitIntegerFactoryStruct(&integerFunctions);
    integerTree = CreateAVLTree(&integerFunctions);

And usage:

    <span style="color:#003366;">struct</span> <span style="color:#008080;">AVL_tree_struct</span>
    {
       <span style="color:#008080;">TreeNode</span>* root;
       <span style="color:#008080;">TreeNode</span>* zero_node;
       <span style="color:#008080;">FuncFactory</span>* functions;
    };
    <span style="color:#003366;">#define</span> <span style="color:#008080;">AVL_tree</span> <span style="color:#003366;">struct</span> <span style="color:#008080;">AVL_tree_struct</span>
    
    <span style="color:#008080;">TreeNode</span>* FindNode(<span style="color:#003366;">const</span> <span style="color:#008080;">AVL_tree</span>* tree, <span style="color:#003366;">const</span> <span style="color:#008000;">void</span>* value)
    {
       <span style="color:#008080;">TreeNode</span>* tn = tree->root;
       <span style="color:#008080;">CompareFunc</span> cmp = tree->functions->compareFunc;
       <span style="color:#003366;">while</span> ( (tn != 0) && ( cmp(tn->value, value) != 0 ))
       {
          <span style="color:#003366;">if</span> (cmp(tn->value, value) > 0)
             tn = tn->left_child;
          <span style="color:#003366;">else</span>
             tn = tn->right_child;
       }
    
       <span style="color:#003366;">if</span> (tn == 0)
          <span style="color:#003366;">return</span> (tree->zero_node);
       <span style="color:#003366;">else</span>
          <span style="color:#003366;">if</span> (IsALeftChild(tn) || IsARightChild(tn))
          {
             <span style="color:#003366;">return</span> tn;
          }
          <span style="color:#003366;">else</span> <span style="color:#003366;">if</span> (tn == tree->root)
                <span style="color:#003366;">return</span> (tree->root);
    
          <span style="color:#003366;">return</span> (tree->zero_node);
    }
