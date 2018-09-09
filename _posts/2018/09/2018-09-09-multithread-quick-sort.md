---
title: 多线程快速排序
date: 2018-09-09
tag:
- C++
- 快速排序
- 多线程
layout: post
excerpt: 多线程执行快速排序（含插入排序信息熵优化）
---

直接上代码喽

```cpp
/**
 * quick_sort.cpp
 * Copyright (c) 2018 Linus Boyle <linusboyle@gmail.com>
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program.  If not, see <http://www.gnu.org/licenses/>.
 */

#include <algorithm>
#include <array>
#include <thread>

static constexpr std::size_t MAX_THREAD = 0;

static std::size_t Thread;

template <class T>
void insertSort(T& array,int left,int right){
    for(int i=left+1;i<=right;++i){
        typename T::value_type key = array.at(i);
        int j = i-1;
        while(j>=left){
            if(array.at(j) > key){
                array.at(j+1) = array.at(j);
            } else {
                break;
            }
            --j;
        }
        array.at(j+1) = key;
    }    
}

template <class T>
void process(T& array,int left,int right){
    if(right-left <= 10){
        insertSort(array,left,right);
        return;
    } 
    
    typename T::value_type pivot = array.at(left); //base 
    int n = left;
    int m = right;

    while(n != m){
        while(array.at(m) >= pivot && n < m)
            m--;
        while(array.at(n) <= pivot && n < m)
            n++;
        if(n < m)
            std::swap(array.at(n),array.at(m));
    }

    std::swap(array.at(left),array.at(n));

    if(Thread>0){
        --Thread;
        std::thread newThread{process<T>,std::ref(array),left,n-1}; 
        process(array,m+1,right); 

        if(newThread.joinable()){
            newThread.join();
        }
    } else {
        process(array,left,n-1);
        process(array,m+1,right);
    }
}

template <class T>
void quicksort(T& array){
    Thread = MAX_THREAD;
    process(array,0,array.size()-1);
}

#include <iostream>
#include <fstream>
#include <random>
#include <boost/timer.hpp>

#define ARRAY_LENGTH 1000000

int main(int argc,char** argv){

    std::random_device rd;
    std::mt19937 mt(rd());
    std::uniform_int_distribution<> dist(0,ARRAY_LENGTH);

    std::array<int,ARRAY_LENGTH> array;

    for(int i = 0;i<ARRAY_LENGTH;++i){
        array.at(i) = dist(mt);
    }
 
    boost::timer timer;
    quicksort(array);
    std::cout<<timer.elapsed()<<" elapsed"<<std::endl;

    //std::fstream out("result.txt",out.trunc|out.out);
    //if(out.is_open())
        //std::for_each(array.begin(),array.end(),[&out](int x){
                //out<<x<<std::endl;
        //});

}
```

关于算法几点说明
- 单枢轴
- 插入排序优化（主要是测试数据量较大）
- 为避免多线程间通信损失，线程数最大为4
