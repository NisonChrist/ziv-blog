---
title: '动态分区分配算法'
description: '操作系统算法实验'
pubDate: 'Jun 18 2024'
heroImage: ''
---

# 动态分区分配算法
>请注意：本次实验代码中有使用的排序方法（sort）属于不稳定排序算法，在最佳适应算法和最坏适应算法中涉及到的可用分区大小的排序皆使用这种不稳定排序，所以其最终的分配方案和通过稳定排序得到的分配方案并不一定相同，但在理论上都是一样可以的

## 实验代码

```cpp
#include <iostream>
#include <vector>
#include <iomanip>
#include <algorithm>

using namespace std;

//分区类
class Partition {
public:
    Partition(int id, int assigned, int available) {
        this->id = id;
        this->assigned = assigned;
        this->available = available;
    }//构造函数

    int id;//分区id
    int assigned;//已分配
    int available;//可分配
};

//进程类
class Process {
public:
    Process(int id, int needPartition, bool isAssigned) {
        this->id = id;
        this->needPartition = needPartition;
        this->isAssigned = isAssigned;
    }//构造函数

    int id;//进程id
    int needPartition;//需要的分区大小
    bool isAssigned;//是否已分配
    int assignedPartitionId{};//被分配到的分区id
};

//分区算法类
class PartitionAlgorithm {
private:
    vector<Partition> partition;//所有分区
    vector<Process> process;//所有进程

public:
    void createPartition();//创建所有分区

    void createProcess();//创建所有进程

    void FF();//首次适应算法

    void NF();//循环首次适应算法

    void BF();//最佳适应算法

    void WF();//最坏适应算法

    void log();//打印结果
};

//创建所有分区
void PartitionAlgorithm::createPartition() {
    int partitionNum;
    cout << "输入分区数：";
    cin >> partitionNum;
    cout << "输入所有分区的大小：";
    for (int i = 0; i < partitionNum; ++i) {
        int partitionSize = 0;
        cin >> partitionSize;
        Partition p(i + 1, 0, partitionSize);
        partition.push_back(p);
    }
}

//创建所有进程
void PartitionAlgorithm::createProcess() {
    int processNum;
    cout << "输入进程数：";
    cin >> processNum;
    cout << "输入所有进程需要分区大小：";
    for (int i = 0; i < processNum; ++i) {
        int needSize = 0;
        cin >> needSize;
        Process p(i + 1, needSize, false);
        process.push_back(p);
    }
}

//首次适应算法
void PartitionAlgorithm::FF() {
    for (auto &i: process) { //循环所有进程
        for (auto &j: partition) { // 每次从头循环所有分区
            if (j.available >= i.needPartition) { //若剩余分区足够分配给当前进程
                j.available -= i.needPartition;// 可分配分区大小减掉分配给当前进程的大小
                j.assigned += i.needPartition;//已分配的大小加上分配给当前进程的大小
                i.isAssigned = true;//将该进程标记为已被分配
                i.assignedPartitionId = j.id;//记录分配到的分区id
                break;
            }
        }
    }
}

//循环首次适应算法
void PartitionAlgorithm::NF() {
    int lastPartitionPos = -1;//记录上一次分区的位置，初始值为-1，因为初次是从0开始
    int count = 0;
    for (auto &i: process) {//循环所有进程
        if (lastPartitionPos == partition.size() - 1) {
            lastPartitionPos = -1;
        }
        for (int j = lastPartitionPos + 1; j < partition.size(); ++j) {//从上一次分区的位置之后开始循环所有分区
            if (partition.at(j).available >= i.needPartition) { //若剩余分区足够分配给当前进程
                partition.at(j).available -= i.needPartition;// 可分配分区大小减掉分配给当前进程的大小
                partition.at(j).assigned += i.needPartition;//已分配的大小加上分配给当前进程的大小
                i.isAssigned = true;//将该进程标记为已被分配
                i.assignedPartitionId = partition.at(j).id;//记录分配到的分区id
                lastPartitionPos = j;//记录结束的分区
                count++;
                break;
            }
            if (j == partition.size() - 1) {//若已经到最后一个分区但仍未找到可以分配的分区
                for (int k = 0; k < lastPartitionPos + 1; ++k) {//则从头开始循环直到上一次分区的位置
                    if (partition.at(k).available >= i.needPartition) {
                        partition.at(k).available -= i.needPartition;
                        partition.at(k).assigned += i.needPartition;
                        i.isAssigned = true;
                        i.assignedPartitionId = partition.at(k).id;
                        count++;
                        break;
                    }
                }
            }
        }
    }
}

//用于sort函数升序排序vector里的剩余分区大小
bool BFCompare(Partition a, Partition b) {
    return a.available < b.available;
}

//用于sort函数降序排序vector里的剩余分区大小
bool WFCompare(Partition a, Partition b) {
    return a.available > b.available;
}

//最佳适应算法
void PartitionAlgorithm::BF() {
    for (auto &i: process) {
        sort(partition.begin(), partition.end(), BFCompare);//先将所有剩余分区大小进行升序排序
        for (auto &j: partition) {
            if (j.available >= i.needPartition) {//若剩余分区足够分配给当前进程
                j.available -= i.needPartition;// 可分配分区大小减掉分配给当前进程的大小
                j.assigned += i.needPartition;//已分配的大小加上分配给当前进程的大小
                i.isAssigned = true;//将该进程标记为已被分配
                i.assignedPartitionId = j.id;//记录分配到的分区id
                break;
            }
        }
    }
}

//最坏适应算法
void PartitionAlgorithm::WF() {
    for (auto &i: process) {
        sort(partition.begin(), partition.end(), WFCompare);//先将所有剩余分区大小进行降序排序
        for (auto &j: partition) {
            if (j.available >= i.needPartition) {//若剩余分区足够分配给当前进程
                j.available -= i.needPartition;// 可分配分区大小减掉分配给当前进程的大小
                j.assigned += i.needPartition;//已分配的大小加上分配给当前进程的大小
                i.isAssigned = true;//将该进程标记为已被分配
                i.assignedPartitionId = j.id;//记录分配到的分区id
                break;
            }
        }
    }
}

//打印结果
void PartitionAlgorithm::log() {
    cout << "------------------------------------------------------------------" << endl;
    const int w = 10;
    cout << left << setw(w) << "分区号";
    for (auto &i: partition) {
        cout << setw(w) << i.id;
    }
    cout << endl;
    cout << left << setw(w) << "已分配";
    for (auto &i: partition) {
        cout << setw(w) << i.assigned;
    }
    cout << endl;
    cout << left << setw(w) << "可分配";
    for (auto &i: partition) {
        cout << setw(w) << i.available;
    }
    cout << endl;
    cout << "------------------------------------------------------------------" << endl;
    cout << left << setw(w) << "进程号";
    for (auto &p: process) {
        cout << setw(w) << p.id;
    }
    cout << endl;
    cout << left << setw(w) << "可分配";
    for (auto &p: process) {
        string yesOrNo = "否";
        if (p.isAssigned)
            yesOrNo = "是";
        cout << setw(w) << yesOrNo;
    }
    cout << endl;
    cout << left << setw(w) << "分区号";
    for (auto &p: process) {
        if (p.isAssigned)
            cout << setw(w) << p.assignedPartitionId;
        else cout << setw(w) << "无";
    }
}

int main() {
    string flag = "y";
    while (flag == "y") {
        PartitionAlgorithm pa;
        pa.createPartition();
        pa.createProcess();
        int choice = 0;
        cout << "选择算法：1-FF, 2-NF, 3-BF, 4-WF: ";
        cin >> choice;
        switch (choice) {
            case 1:
                pa.FF();
                break;
            case 2:
                pa.NF();
                break;
            case 3:
                pa.BF();
                break;
            case 4:
                pa.WF();
                break;
            default:
                cout << "输入不合法！";
                break;
        }
        pa.log();
        cout << endl << "是否继续尝试其他算法？y/n: ";//输入y为继续，n退出（其他任意值也可退出）
        cin >> flag;
    }
}

```
