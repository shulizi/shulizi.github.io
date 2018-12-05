---
layout: post
title:  "C++修炼篇：操作系统进程调度"
date:   2018-12-04
categories: cpp
tags: cpp process pcb
---

* content
{:toc}




## 代码

MyProcessControl.cpp
``` java
include "ProcessStruct.h"
#include <iostream>
#include <queue>
using namespace std;
#ifndef MyProcessControl_h
#define MyProcessControl_h
class MyProcessControlClass{
private:
ProcessStruct* run;
ProcessStruct* head;
ProcessStruct* tail;
int lengthOfPBP = 0;
ProcessStruct* processesByPriority[10];
queue<ProcessStruct*> finishedQueue;

public:
void createProcesses();
void startProcesses();
void showProcesses();
void insertProcessByPriority(ProcessStruct*);
};
void MyProcessControlClass::createProcesses(){

ProcessStruct* p1 = new ProcessStruct(1,6,4);
ProcessStruct* p2 = new ProcessStruct(2,6,2);
ProcessStruct* p3 = new ProcessStruct(3,4,4);
ProcessStruct* p4 = new ProcessStruct(4,4,5);
ProcessStruct* p5 = new ProcessStruct(5,8,10);
ProcessStruct* p6 = new ProcessStruct(6,4,2);

insertProcessByPriority(p1);
insertProcessByPriority(p2);
insertProcessByPriority(p3);
insertProcessByPriority(p4);
insertProcessByPriority(p5);
insertProcessByPriority(p6);

};
void MyProcessControlClass::insertProcessByPriority(ProcessStruct* p){

p->nextStruct = p;
if (lengthOfPBP == 0){
processesByPriority[0] = p;
lengthOfPBP++;
return;
}
int i = 0;
for (;i < lengthOfPBP;i++){

if(p->priority > processesByPriority[i]->priority){

for (int j = lengthOfPBP;j > i;j--){
processesByPriority[j] = processesByPriority[j-1];
}
processesByPriority[i] = p;
lengthOfPBP++;
break;
}else if (p->priority == processesByPriority[i]->priority){
p->nextStruct = processesByPriority[i]->nextStruct;
processesByPriority[i]->nextStruct = p;
break;
}

}

if (i == lengthOfPBP){

processesByPriority[lengthOfPBP] = p;
lengthOfPBP++;
}
};
void MyProcessControlClass::startProcesses(){
while (lengthOfPBP != 0) {
showProcesses();
run = processesByPriority[0];
while (true){

run->timeToFinish --;
run->priority -= 3;
if (run->timeToFinish == 0){

finishedQueue.push(run);
if(processesByPriority[0] -> nextStruct != processesByPriority[0]){
processesByPriority[0]->nextStruct = processesByPriority[0]->nextStruct->nextStruct;
} else {
for (int i = 0;i < lengthOfPBP - 1;i++){
processesByPriority[i] = processesByPriority [i+1];
}
lengthOfPBP--;
}

break;
} else if(lengthOfPBP > 1 && run->priority < processesByPriority[1]->priority){

for (int i = 0;i < lengthOfPBP - 1;i++){
processesByPriority[i] = processesByPriority [i+1];
}
lengthOfPBP--;
insertProcessByPriority(run);

break;
} else{
processesByPriority[0] = processesByPriority[0]->nextStruct;
run = processesByPriority[0];
}

}
}
};

void MyProcessControlClass::showProcesses(){
cout << "----------------" << endl;
cout << "id priority tToF" << endl;
for (int i = 0;i < lengthOfPBP;i++){
ProcessStruct* temp = processesByPriority[i];
ProcessStruct* head = processesByPriority[i];
do {
cout << temp->id << " ";
cout << temp->priority << " ";
cout << temp->timeToFinish << " " << endl;
temp = temp->nextStruct;

}while (temp != head);
cout << "-----" << endl;
}

}



#endif /* MyProcessControl_h */

```
ProcessStruct.cpp
``` java
//
//  ProcessStruct.h
//  ProcessControl
//
//  Created by 杜李 on 2018/12/1.
//  Copyright © 2018年 杜李. All rights reserved.
//

#ifndef ProcessStruct_h
#define ProcessStruct_h
struct ProcessStruct{
int id;
int priority;
int timeOnCPU;
int timeToFinish;
char state;
ProcessStruct* nextStruct;
ProcessStruct(int,int,int);
};
ProcessStruct::ProcessStruct(int id,int priority,int timeToFinish){
this->id = id;
this->priority = priority;
this->timeToFinish = timeToFinish;
this->timeOnCPU = 0;
this->state = 'W';
this->nextStruct = NULL;
}


#endif /* ProcessStruct_h */

```
main.app
``` cpp
//
//  main.cpp
//  ProcessControl
//
//  Created by 杜李 on 2018/11/30.
//  Copyright © 2018年 杜李. All rights reserved.
//

#include <iostream>
#include <queue>
#include "ProcessStruct.h"
#include "MyProcessControl.h"
using namespace std;




class PriorityClass{
private:
ProcessStruct* run;
ProcessStruct* head;
ProcessStruct* tail;
queue<ProcessStruct*> finishedQueue;

public:
void createProcesses();
void startProcesses();
void showProcesses();
void insertProcessByPriority(ProcessStruct*);
};

void PriorityClass::createProcesses(){
head = new ProcessStruct(1,3,5);
ProcessStruct* p2 = new ProcessStruct(2,6,2);
ProcessStruct* p3 = new ProcessStruct(3,2,4);
ProcessStruct* p4 = new ProcessStruct(4,4,5);
ProcessStruct* p5 = new ProcessStruct(5,8,4);

insertProcessByPriority(p2);
insertProcessByPriority(p3);
insertProcessByPriority(p4);
insertProcessByPriority(p5);

}
void PriorityClass::insertProcessByPriority(ProcessStruct* p){
ProcessStruct* temp = head;
ProcessStruct* preStruct = NULL;

while (temp != NULL) {

if(p->priority > temp->priority){
if(preStruct == NULL){
p->nextStruct = head;
head = p;
} else{
preStruct->nextStruct = p;
p->nextStruct = temp;
}
break;
}
preStruct = temp;
temp = temp->nextStruct;

}
if(temp == NULL){
preStruct -> nextStruct = p;
tail = p;
}
}
void PriorityClass::startProcesses(){
while (head != NULL) {
showProcesses();
run = head;
while (true){
run->priority -= 3;
run->timeToFinish--;
if (run->timeToFinish == 0){

finishedQueue.push(run);
head = head->nextStruct;
break;
} else if(head->nextStruct!=NULL && run->priority < head->nextStruct->priority){

head = head->nextStruct;
run->nextStruct = NULL;
insertProcessByPriority(run);

break;
}

}
}
}
void PriorityClass::showProcesses(){
cout << "--------" << endl;
cout << "id priority tToF" << endl;
ProcessStruct* temp = head;
while (temp != NULL) {
cout << temp->id << " ";
cout << temp->priority << " ";
cout << temp->timeToFinish << " " << endl;
temp = temp->nextStruct;

}
}









class RoundRobinClass{
private:
ProcessStruct* run;
ProcessStruct* head;
ProcessStruct* tail;
queue<ProcessStruct*> finishedQueue;

public:
void createProcesses();
void startProcesses();
void showProcesses();
};
void RoundRobinClass::createProcesses(){
head = new ProcessStruct(1,3,5);
ProcessStruct* p2 = new ProcessStruct(2,6,2);
ProcessStruct* p3 = new ProcessStruct(3,2,4);
ProcessStruct* p4 = new ProcessStruct(4,4,5);
ProcessStruct* p5 = new ProcessStruct(5,8,4);

head->nextStruct = p2;
p2->nextStruct = p3;
p3->nextStruct = p4;
p4->nextStruct = p5;

tail = p5;


}
void RoundRobinClass::startProcesses(){
while (head != NULL) {
showProcesses();
run = head;
while (true){
run->timeOnCPU++;
run->timeToFinish--;
if (run->timeToFinish == 0){

finishedQueue.push(run);
head = head->nextStruct;
break;
} else if(run->timeOnCPU == run->priority){

run->timeOnCPU = 0;
tail->nextStruct = run;
tail = run;

head = head->nextStruct;
tail->nextStruct = NULL;
break;
}
}
}
};
void RoundRobinClass::showProcesses(){
cout << "--------" << endl;
cout << "id priority tToF" << endl;
ProcessStruct* temp = head;
while (temp != NULL) {
cout << temp->id << " ";
cout << temp->priority << " ";
cout << temp->timeToFinish << " " << endl;
temp = temp->nextStruct;

}
};
int main(){
cout << "hello world!" << endl;
//    RoundRobinClass roundRobin;
//    roundRobin.createProcesses();
//    roundRobin.startProcesses();
//    PriorityClass priorityC;
//    priorityC.createProcesses();
//    priorityC.startProcesses();
MyProcessControlClass myProcessControl;
myProcessControl.createProcesses();
myProcessControl.startProcesses();
myProcessControl.showProcesses();

return 0;
}

```

