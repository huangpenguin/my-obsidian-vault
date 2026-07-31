---
title: "高速化(vectorize)"
publish: false
tags: ["待整理"]
---
# 高速化(vectorize)

```matlab
function rev = delay_line(Y, delay0, delay1, delay2, delay3, dev, out, M, N)
    delay0_else =  mod(delay0+1,M)+1;
    delay1_else =  mod(delay1+1,M)+1;
    delay2_else =  mod(delay2+1,M)+1;
    delay3_else =  mod(delay3+1,M)+1;

    for i = 1:N
        if dev(i)<1
        	out(i)=interpl_4(Y(i, delay0(i)+1), Y(i, delay1(i)+1) ,Y(i, delay2(i)+1),Y(i, delay3(i)+1),dev(i));
        else
            out(i)=interpl_4(Y(i, delay0_else(i)),Y(i, delay1_else(i)), Y(i, delay2_else(i)),Y(i, delay3_else(i)),dev(i)-1);
        end
    end
    rev = out;
end

function rev = interpl_4(a, b, c, d, frac)
    cminusb = c-b;
    rev = b + frac * (cminusb - 0.166666667 * (1.-frac) * ((d - a - 3.0 * cminusb) * frac + (d + 2.0*a - 3.0*b)));
end
```

```matlab
%ベクトル化
function rev = delay_line(Y, delay0, delay1, delay2, delay3, dev, out, M, N)

delay0_else = mod(delay0 + 1, M) + 1;
delay1_else = mod(delay1 + 1, M) + 1;
delay2_else = mod(delay2 + 1, M) + 1;
delay3_else = mod(delay3 + 1, M) + 1;

y0 = Y(sub2ind(size(Y), 1:1001, delay0 + 1));  
y1 = Y(sub2ind(size(Y), 1:1001, delay1 + 1)); 
y2 = Y(sub2ind(size(Y), 1:1001, delay2 + 1));  
y3 = Y(sub2ind(size(Y), 1:1001, delay3 + 1)); 

y0_else = Y(sub2ind(size(Y), 1:1001, delay0_else)); 
y1_else = Y(sub2ind(size(Y), 1:1001, delay1_else)); 
y2_else = Y(sub2ind(size(Y), 1:1001, delay2_else)); 
y3_else = Y(sub2ind(size(Y), 1:1001, delay3_else)); 

dev_less_than_1 = dev < 1;

out = zeros(1, N);  
out(dev_less_than_1) = interpl_4(y0(dev_less_than_1), y1(dev_less_than_1), y2(dev_less_than_1), y3(dev_less_than_1), dev(dev_less_than_1));
out(~dev_less_than_1) = interpl_4(y0_else(~dev_less_than_1), y1_else(~dev_less_than_1), y2_else(~dev_less_than_1), y3_else(~dev_less_than_1), dev(~dev_less_than_1) - 1);

rev = out;
end

```

注意在使用profile比较速度的时候，由于只能显示运行时的时间分布情况，比较方便的做法是把**老代码也拷进来一起运行**，在不超内存的情况下比较时间的分布，可以看出是否做到优化。

但是注意profile测出来的时间不一定是对的，这次就与使用tictoc得到的运行时间大相径庭,由于循环次数过多可能存在***overhead***的情况。
