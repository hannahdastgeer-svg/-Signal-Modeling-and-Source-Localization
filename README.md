# -Signal-Modeling-and-Source-Localization
Simulating the received signals, estimating its locations, automating the estimates, modeling impact of noise 
s = @(t) 1000 * cos(880*pi*t) .* heaviside(t);
A = 0.5; B = 100; L = 100;
cs = 333+1/3;
delay1 = sqrt(B^2 + (L-A)^2)/cs; delay2 = B/cs;
fs = 44.1e3;
a = min(delay1, delay2) - 0.01; b = max(delay1, delay2) + 0.01;
t = (min(delay1, delay2) - 0.01):(1/fs):(max(delay1, delay2) + 0.01);

%Received signals
y1 = s(t-delay1);
y2 = s(t-delay2);

%Simulate sending this s(t) over the channel using fs = 44.1kHz
figure;
plot(t, s(t));
title('s(t)')
xlabel('Time (s)')
ylabel('Amplitude')
<img width="1157" height="746" alt="Figure_1" src="https://github.com/user-attachments/assets/0da9c308-c145-478b-b6bc-cef2c72c97c9" />

%Plot y1(t) and y2(t) using subplot
figure;
subplot(2,2,1)
plot(t,y1)
title('y1(t)')
xlabel('Time (s)')
ylabel('Amplitude')
subplot(2,2,2)
plot(t,y2)
title('y2(t)')
xlabel('Time (s)')
ylabel('Amplitude')
<img width="1149" height="358" alt="Figure_2" src="https://github.com/user-attachments/assets/dd6f76e4-ddd3-46a0-a5f3-84e1617d76aa" />

%Use lab1sim to generate y1(t) and y2(t) for t € [0, 0.5].
fs = 44.1e3;
t = 0:(1/fs):0.5; % Define the time vector
[y1, y2] = lab1sim(0.5,100,100,@(t) 1000 * cos(880*pi*t) .* heaviside(t));

%Use the xcorr function to plot the cross-correlation between y1 (t) and y2(t)
[C, lags] = xcorr(y1(t), y2(t));
figure;
plot(lags/fs, C);
xlabel('Time (s)');
ylabel('Cross-Correlation');
title('Cross-Correlation between y1(t) and y2(t)');
grid on;
<img width="1134" height="747" alt="Figure_3" src="https://github.com/user-attachments/assets/0e1eafdf-78a5-424c-b17a-32b9a6fe9760" />

%Use the max function to find the peak in the autocorrelation and convert the corresponding lag into an estimate of the relative time shift
peak_C = max(C);
[~, peakIndex] = max(C);
relativeTimeShift = lags(peakIndex) / fs;

%Use linspace to generate 100 evenly spaced points spaced between L = 1 meter and L = 100 meters.
L_values = linspace(1, 100, 100);

%Write a loop that generates signals from a speaker at location L using lab1sim() and then estimates L using lab1est().
estimatedL = zeros(1, length(L_values));
for x = 1:length(L_values)
    [sig1,sig2] = lab1sim(0.5,100,100,@(x) 1000 * cos(880*pi*x) .* heaviside(x));
    estimatedL(x) = lab1est(0.5,100,sig1,sig2);
end

%Plot the relative error between the true L and the estimate (which we can call Î):
Erel = abs((L_values - estimatedL) ./ L_values) * 100; % Calculate relative error
figure;
plot(L_values, Erel);
xlabel('True L (m)');
ylabel('Relative Error (%)');
title('Relative Error between True L and Estimated L');
grid on;
<img width="1135" height="746" alt="Figure_4" src="https://github.com/user-attachments/assets/20a6142a-716c-4aa1-bb8e-0212abf3c0a9" />

% Plot the estimated L values against the true L values
figure;
plot(L_values, estimatedL, 'o-');
xlabel('True L (m)');
ylabel('Estimated L (m)');
title('Estimated L vs True L');
grid on;
<img width="1129" height="746" alt="Figure_5" src="https://github.com/user-attachments/assets/05df8edb-e26d-4371-84d6-23bf84002af1" />

%Generate two independent random vectors of unit-variance Gaussian noise wi and w2 using randn.
%Continue generating signals only for t e [0,0.5] and L = 100.
t = 0:(1/fs):0.5;
w1 = randn(t); w2 = randn(t);

alpha = linspace(10,150,10);
noisy_signal = zeros(1, length(alpha));

for n = 1:length(alpha)
    [noisy1,noisy2] = lab1sim(0.5,100,100,@(n) 1000 * cos(880*pi*n) .* heaviside(n));
    estimatedL(n) = lab1est(0.5,100,noisy1,noisy2);
end

