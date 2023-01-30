---
layout:     post   				    # 使用的布局（不需要改）
title:      数据结构 				# 标题 
subtitle:   单链表基本操作    #副标题
date:       2023-01-30 				# 时间
author:     Derrick 				# 作者
header-img: img/post-bg-dataStruct.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - C语言
---

### 单链表的基本操作

<br/><br/>
```c
#include <iostream>
using namespace std;
#define MaxSize 10 
typedef struct {
    int data[MaxSize];
    int length;
}SqList;

//初始化表
void InitList(SqList &L)
{
    L.length=0;
}

//插入操作
bool ListInsert(SqList &L , int index , int e){ // 插入的顺序表，位置，元素 
    if(index < 1 || index > L.length+1)/// id 是元素的位置，下表是从1开始的
        return false; // 插入失败
    if(L.length >= MaxSize)
        return false;
    for(int i = L.length ; i >= index ; i--){
        L.data[i] = L.data[i-1];
    } 
    L.data[index-1] = e;
    L.length ++;
    return true; // 插入成功  
}

//删除操作
bool ListDelete(SqList &L , int index , int &e){
    // e 为删除的变量 ， 把这个变量“带回去”，所以加个& ，
    //不用int 是这里的bool型函数用来检查有没有删除成功，
    //改成int返回的话，就不用定义e了，但是不能判断删除操作是成功了 
    if(index < 1 || index > L.length)
        return false;
    e = L.data[index-1]; // 保存删除的元素
    for(int i = index; i < L.length ; i++){
        L.data[i-1] = L.data[i];
    } 
    L.length--;
    return true;
     
}

//按值查找操作
int LocateElem(SqList L, int e){// 查找顺序表中第一个等于e的位序（不是下标） 
    for(int i = 0 ; i < L.length ; i++){
        if(L.data[i] == e)
            return i+1 ;
    }
    return -1;
}

//按位查找
int GetElem(SqList L, int i)
{
    return L.data[i-1];
}

//////////////////////////////////////////////////////////////////////////////////
int main()
{
    /// 这里一定不要搞混下标和位置 ， 数组下是从0开始的，位置是从1开始的 

    SqList L;
    InitList(L);
    L.data[0] = 1 ; 
    L.data[1] = 2 ;
    L.data[2] = 4 ;
    L.data[3] = 5 ;
    L.data[4] = 6 ;    
    L.length = 5; // 插入数据后要改长度， 不要马大哈 
    
    //顺序表初始状态
    for(int i = 0;i < L.length;i++) 
    {
        printf("%d", L.data[i]); 
    }
    printf("\n");

    
    //插入操作
    if(ListInsert(L,3,3))
    {
        printf("success!\n");
    }
    else
    {
        printf("failed!\n");
    }
    
    
    printf("查询第一个等于e的位序：%d\n",LocateElem(L,3));//e 为 2
    printf("查询第3位序的值：%d\n",GetElem(L,3));//查询顺序表中第5位


    int e = -100000; // 用来接收返回的删除掉的元素 
    if(ListDelete(L,3,e)){
        
        printf("Delete succeed!\n");

        printf("Delete element is %d\n" , e);
        for(int i = 0;i < L.length;i++) 
        {
            printf("%d" , L.data[i]); 
        }
            printf("\n");
    } 
    else{
        printf("Delete fail!\n");
    }
    

```

