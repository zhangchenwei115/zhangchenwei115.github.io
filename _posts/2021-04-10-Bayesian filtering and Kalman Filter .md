---
layout:     post
title:      Bayesian filtering and Kalman Filter 
subtitle:   The code of Kalman Filter with Matlab
date:       2021-04-10
author:     Hi
header-img: img/art-Anaconda-TensorFlow.jpg
catalog: true
tags:
    - Kalman Filter

---


# Note
https://drive.google.com/file/d/1oVYTk0dHv8rTWhI0g_ioi-o6MwtvwuZd/view?usp=sharing

# code
```bash
%%%kalman filter
% X(K) = F*X(X-1)+Q
% Y(K) = H*X(X)+R
%%%第一个问题，生成一段随机信号，并滤波

% 生成一段时间,L代表采样了多少次
t=0.1:0.01:1;
L=length(t);
%生成真实信号x，以及观测y
%首先初始化
x=zeros(1,L);
y=x
%生成信号，设x=t^2，（没有传感器所以就用数学生成）
for i=1:L
    x(i)=t(i)^2
    y(i)=x(i)+normrnd(0,0.1);%正态分布，参数为均值和标准差
end
%%%%信号生成完毕%%%%%%%%%%
%%%%%%%%%%滤波算法%%%%%%
%plot(t,x,t,y,'LineWidth',2)  %%解释，蓝色的为真实的准确信号，我们不知道，我们需要通过橘色歪歪扭扭的有噪音的信号，滤波以后得到蓝色的。
%%%预测方程和观测方程怎么写%%%
%%观测方程好些 Y(K)=X(K)+R R N~(0,1) 最麻烦的就是Q和R怎么选，没啥办法，只能试，选。调参
%预测方程不好写，一堆毫无规则的曲线%%%在这里可以猜一猜是线性增长，但大多数问题，信号时杂乱无章的。
%预测模型就是猜的
%X(K)=X(K-1)+Q F1=1
%Y(K)=X(K)+R H1=1
%猜 Q~N(0,1)
%%%%%开始滤波
F1=1;
H1=1;
Q1=1;
R1=1;
%初始化x(k)+ Pplus,Pminus,P指的是协方差，方差！！！
Xplus1=zeros(1,L);
%设置一个处置，假设Xplus1(1)~(0.01,0.01^2) 应为第一个数就是0.01，P方差可以小一点，自信，但可以调

Xplus1(1)=0.01;
Pplus1=0,01^2;

%%卡尔曼滤波算法.5个公式
%X(K)minus=F*X(K-1)plus 
%P(K)minus=F*P(K-1)plus*F`+Q
%K=P(K)minus*H`*inv(H*P(K)minus*H`+R)
%X(K)plus=X(K)minus+K*(y(k)-H*X(K)minus)
%P(K)plus=(1-K*H)*P(K)minus
for i=2:L
    %注意为啥代码和上面的不太一样，P，因为每个循环都更新，就P就不需要数组存了
    %%预测步
    Xminus1=F1*Xplus1(i-1);
    Pminus1=F1*Pplus1*F1'+Q1;
    %更新步
    K1=(Pminus1*H1')*inv(H1*Pminus1*H1'+R1);
    Xplus1(i)=Xminus1+K1*(y(i)-H1*Xminus1);
    Pplus1=(1-K1*H1)*Pminus1;
end

%plot(t,x,t,y,t,Xplus1,'LineWidth',2);
%%模型2,比上面的精细一点
%X(K)=X(K-1)+X'(K-1)*dt+X''(K-1)dt^2*(1/2!)+Q2 预测方程
%Y(K)=X(K)+R R~N(0,1)
%此时状态变量X=[X(K) X'(K) X''(K)]T(列向量)
%Y(K)=H*X+R  H=[1 0 0]行向量
%预测方程
%X(K)=X(K-1)+X'(K-1)*dt+X''(K-1)dt^2*(1/2!)+Q2
%X'(K)=0*X(K-1)+X'(K-1)+X''(K-1)dt+Q3
%X''(K)=0*X(K-1)+0*X'(K-1)+X''(K-1)dt+Q4  %%%多项式多求几次导数，总会比较平缓，而
%X''(K)=X''(K-1)dt+Q4 正是描述平缓的随机过程，这种建模相对精细一些，适用范围也较广
%F1=1 dt 0.5*dt^2
%   0  1   dt
%   0  0   1
%F就是预测函数的参数！！！这里是个矩阵，上面推导出来的
%H=[1 0 0]
%Q= Q2  0   0
%   0   Q3  0
%   0   0   Q4
%我的理解下面的q2和上面的q是一个意思。
dt=t(2)-t(1);
F2=[1,dt,0.5*dt^2;0,1,dt;0,0,1];
H2=[1,0,0]
Q2=[1,0,0;0,0.1,0;0,0,0.0001]
R2=20;
%%%设置初值
Xplus2=zeros(3,L);
Xplus2(1,1)=0.1^2;
Xplus2(2,1)=0;
Xplus2(3,1)=0;
Pplus2=[0.01,0,0;0,0.01,0;0,0,0.0001]
for i=2:L
    Xminus2=F2*Xplus2(:,i-1);
    Pminus2=F2*Pplus2*F2'+Q2;
    K2=(Pminus2*H2')*inv(H2*Pminus2*H2'+R2);
    Xplus2(:,i)=Xminus2+K2*(y(i)-H2*Xminus2);
    Pplus2=(eye(3)-K2*H2)*Pminus2;  %eye 单位矩阵
end
plot(t,x,'r',t,y,'g',t,Xplus1,'b',t,Xplus2(1,:),'m','LineWidth',2);
%很牛
%问题2，两个传感器，进行滤波

% Y1(K)=X(K)+R

% Y2(K)=X(K)+R

%H=[1 1]T (列向量) X=X(K)

%H=1   0     0     X=X(K)   X'(K)   X''(K)

%     1   0     0

    

F3=[1,   dt,   0.5*dt^2;0,   1,   dt;0,   0,     1];%%%此处要注意矩阵是否病态，若dt特别小，易导致矩阵病态或精度丢失

H3=[1,0,0;1,0,0];

Q3=[1,  0,  0;0,  0.01,  0;0,  0,  0.0001];

R3=[3,0;0,3];%%%%%一定要注意是协方差矩阵

%%%设置初值%%%%

Xplus3=zeros(3,L);

Xplus3(1,1)=0.1^2;

Xplus3(2,1)=0;

Xplus3(3,1)=0;

Pplus3=[0.01, 0,   0;0,    0.01,   0;0,   0,    0.0001];

for i=2:L

    %%%预测步%%%

    Xminus3=F3*Xplus3(:,i-1);

    Pminus3=F3*Pplus3*F3'+Q3;

    %%%更新步%%%%

    K3=(Pminus3*H3')*inv(H3*Pminus3*H3'+R3);

    Y=zeros(2,1);

    Y(1,1)= y(i);

    Y(2,1)=y2(i);

    Xplus3(:,i)=Xminus3+K3*(Y-H3*Xminus3);

    Pplus3=(eye(3)-K3*H3)*Pminus3;

end

```
![picture1](/img\kalmanfilter.jpg)  
* 粉红色为模型2 更精确的滤波，绿色为观察值，红色为真实值，蓝色为模型1的滤波