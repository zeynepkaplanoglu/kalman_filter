## Description

Kalman Filter is a filter that can predict the state of the system from the input and output information together with the previous information of the model in a dynamic system represented by the state space model. 
The Kalman Filter is an algorithm used after the 1960s, mainly in vehicle navigation (although aerospace applications are typical, but also in other application areas), providing an optimized estimate of the state of the system. The algorithm works on a noisy observation data stream (typically sensor measurements) in real-time, recursively, filtering to minimize error and optimize it according to the mathematical prediction of the future state generated by modeling the physical characteristics of the system.
We can think of the mathematical model as a function. Functions have parameters (input) and return values (output). Building the mathematical model can sometimes be very difficult. It is very difficult for a linear system to model a non-linear system, although sometimes it is easy. You cannot reflect the real system exactly as a mathematical model. You should try to create the mathematical model closest to the real system. Because the data we obtain will be noisy and inaccurate data. In real life, there may be factors affecting our system that we are not aware of and modeling all of these factors will be difficult and confusing.
Since our goal of this tutorial is to implement the Kalman filter in computer programming code, this tutorial was created exclusively for the Discrete Kalman filter.
The basic idea of the Kalman filter is to use previous knowledge of the situation, the filter makes a forward projection state or predicts the next situation.
The Kalman Filter aims to predict the state of a system at time k using the linear stochastic difference equation, assuming that the state of a system at time k has evolved as written from the previous state at time k-1.
## Prerequisites
MATLAB 
## Folder Structure
Kalman_filter

|── sample_input

|   |── matlab kalman formula, moving object data, noise

|── sample_output

|   |── time and position graph, filtered output graph

|── example.py

|── Time_and_Position_for_Moving_Object.py

|── parameter_table.py

|── kalman_filter.txt

|── LICENSE

## Example

This example is created with MATLAB.
In this example, we want to model a moving object that follows a simple path as given in the pos function.

## Time and Position for Moving Object 

```
clear all;
clc;

t = linspace(0,100,1001);%Discrete times from 1 to 100 for time t

%model a moving object that follows a simple path as given in the following function:
pos = 0.1*(t.^2-t);


sz = size(pos); %the variable named sz takes "position matrix" (pos) size
meas = zeros(sz(2),1); %zeros vector of size 101x1 is created by using sz variable to initialize measurement vector

for i=1:sz(2)
    meas(i) = pos(i) + (rand(1)-0.5)*500; 
end

% previously created empty measurement vector, meas, is filled in the above
% for loop
% meas vector is based upon "position vector" (pos). In each iteration a
% random value is added to value of position vector corresponding
% measurement value. 

%plot(t,pos) %position value vs dicrete time plot

dt = 0.1; %discrete time value
sigma_acc = 0.25; %standard devitaion of acceleration
sigma_meas = 1.2; %standard deviation of measurement

A = [1 dt; 0 1]; %State transition matrix A
B = [0.5*dt^2; dt]; %control matrix B

H = [1 0]; %Transformation matrix H

Q = [dt^4/4 dt^3/2; dt^3/2 dt^2]*sigma_acc^2; %Proess noise covariance matrix Q

R = sigma_meas^2; %Measurement noise covariance matrix R

p0 = eye(2); 

x0 = [0; 0];

u = 2;
```

## Parameter Table
```

KM = cell(1002,7); %Struct variable of size 102x7


% P0 = 10; %P için tahmin 
% x0 = 10; %x için tahmin

% 

%KM is a struct variable holding Ppe "Xpe P X K z" pos for its columns

KM(1,4) = {x0};
KM(1,3) = {p0};

pos = pos';

for i=2:1002
    KM(i,6) = {meas(i-1,1)};
    KM(i,7) = {pos(i-1,1)};
end

for i=2:1002
    KM(i,1)={A*cell2mat(KM(1,4))};
end
```
## Kalman Filter

```

for i=2:1002

    KM(i,2)={A*cell2mat(KM(i-1,4)) + B*u}; %priori state estimates
    KM(i,1)={A*cell2mat(KM(i-1,3))*A'+Q};  %priori error covariance
    
    Ppe = cell2mat(KM(i,1)); %cells converted to matrix
    Xpe = cell2mat(KM(i,2)); %cells converted to matrix
    
    KM(i,5)={Ppe*(H'*inv(H*Ppe*H'+R))}; %Calculation of Kalman Gain
   
    K = cell2mat(KM(i,5)); %Kalman Gain
    z = cell2mat(KM(i,6)); %measured position
    
    KM(i,4)={Xpe+K*(z-H*Xpe)}; %update estimate with measurement
    
    KM(i,3)={(eye(2)-K*H)*Ppe}; %update the error covariance

end


kal1 = KM(2:1002, 4);
kalm_mat = cell2mat(kal1);
kalm_mat_res = reshape(kalm_mat, [2,1001]);
pred_pos = kalm_mat_res(1,:); %predicted position vector
pred_pos = pred_pos'; %predicted position vector transpose

plot(t, pos, "Color", "green", "LineWidth",2);
hold on;
plot(t, meas,"Color","red");
hold on;
plot(t, pred_pos, "Color", "blue", "LineWidth",1, "LineStyle","-.")

legend("actual", "measured", "kalman_pred")

hold off;

clf('reset')

```
