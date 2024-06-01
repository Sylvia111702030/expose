# 期末考

# Matlab 程式

## 作業

### 作業1

<aside>
<img src="https://www.notion.so/icons/defibrillator_gray.svg" alt="https://www.notion.so/icons/defibrillator_gray.svg" width="40px" /> 產生一筆個人實驗模擬資料，以數值矩陣(命名為data, 60x6) 的形式存放附表的欄位數值

</aside>

- 此為受試者編號為1的實驗資料，共有60個嘗試次
- 二個獨變項，分別為顏色(紅、藍、綠)和形狀(圓形、方形)，各情境(共六種組合)的嘗試次數量為10
- 情境順序隨機安排且出現的嘗試次數量相同，前後半嘗試次(各30)的各情境出現次數相等。
- 反應時間用隨機數值模擬產生，介於400-800之間;反應正確用隨機數值模擬產生，正確為1，錯誤為0，整體正確率為85%

**程式碼**

```matlab
clearvars;
id = ones(60,1);
tno = 1:60;
% 產生30x1 shape [1 1...1 2 2...2]
shape0 = repmat([1 2],15,1);
shape = shape0(:); % 轉置
% 產生30x1 color [1 2 3 ... 1 2 3]
color = repmat([1:3]',[10 1]);
% 組合 shape 和 color
data0 = [shape color];
% 產生兩個介於1-30的隨機順序數列
rmtx = rand(30,2);
[~,rseq] = sort(rmtx);
% 產生兩組打亂shape和color的30x2矩陣並相接成60x2的矩陣
data0a = data0(rseq(:),:);
% 產生60x1 rt 介於400 ～800
rt = rand(60,1)*400 + 400;
% 產生60x1 acc 85% 為 1 並打亂順序
acc0 = zeros(60,1);
acc0(1:60*.85) = 1;
rmtx = rand(60,1);
[~,rseq] = sort(rmtx);
acc = acc0(rseq);
% 合併所有變項產生 data
data = [id tno data0a rt acc];
```

**重要概念**

1. 轉置的方法：[x]'，若為n * 1的矩陣可以直接用x(:)
2. [~,rseq] = sort(rmtx); 意指讓 rseq 依據 rmtx 的大小順序對應
    1. 例如 rmtx 為0.75、0.25、0.50、0.69，那rseq就會是2、3、4、1（最小的是rmtx裡的第二個）
3. acc = acc0(rseq); 代表讓acc0根據rseq的方式排
    1. 若原始的acc0為1、1、1、0，則acc為1、1、0、1（第四個數字0，會因為resq的第三個是4，而排到acc的第三位置）

### 作業2

<aside>
<img src="https://www.notion.so/icons/defibrillator_gray.svg" alt="https://www.notion.so/icons/defibrillator_gray.svg" width="40px" /> 在程式中執行 ‘hw1_學號’後，排除答錯的嘗試次，用**迴圈**來計算各情境的反應時間平均值、標準差、有效觀察值個數，存於stat_res變項(6x6, 包括id,shape,color, 及三項統計值)

</aside>

**程式碼**

```matlab
cleavers;
% clear the display of command window
clc;
% run hw1 to get the data
hw1
% generate all zero matrix for statistics
stat_res = zeros(6,6);
% 選擇正確的資料
fdata = data(data(:,6)==1,:);
for color = 1:3
    for shape = 1:2
        % get the RTs for each condition
        rt = fdata(fdata(:,3)==shape & fdata(:,4)==color,5);
        % 新的資料表格中，資料排列為shape1color1、shape1color2、......
        rowno = (shape-1)*3 + color;
        % put the mean std and n to the row of stat_res
        stat_res(rowno,:) = [fdata(1,1) shape color mean(rt) std(rt) length(rt)];
    end
end
% show the names of columns in stat_res
fprintf(' id   shape color    RT      SD    N\n');
% format and show the statistics
fprintf('%3d     %d     %d   %7.3f %7.3f %2d\n',stat_res');
```

**重要概念**

1. for 迴圈，迴圈內的東西要記得宣告
2. 條件選擇要用==（或是> =或< = ）因為單個等號是宣告變數
3. fprintf的模式：先設定格式，後填上資料
    1. d是一般的數字double
    2. f是有小數的時候用，%7.3f 代表留小數後到第三位，小數點前有4個位置（未滿四位會自己空著）
    3. 補充：s是字串

### 作業3

<aside>
<img src="https://www.notion.so/icons/defibrillator_gray.svg" alt="https://www.notion.so/icons/defibrillator_gray.svg" width="40px" /> Navon 實驗

- 實驗設計：
    - 自變項：作業要求、一致與否
    - 依變項：反應時間、正確率
- 資料分析排除
    - 反應時間：超時、小於400毫秒或大於800毫秒、答錯
    - 正確率：超時
- 用迴圈計算每一位受試者在各情境的統計值：
    - 將每位受試者的ID依操弄變項的組合，計算各情境：有效嘗試次數量（排除超時）(ntrial)、有效嘗試的正確率(CR)、有效且正確的反應時間平均值(RT)。
    - 結果存放於stats(...x7) 的表格
- 計算各情境的整體統計數值
    - 在螢幕以合適的格式呈現 (固定欄位寬度和小數位數）呈現總受試人數(N)、操弄變項之情境(task, cong)、及各水準下的總平均值：包括嘗試次(ntrials)、正確率(CR)、及反應時間(RT） [共六欄和四行資料]
</aside>

**程式碼**

```matlab
clearvars;

% PART 1
gtotal = 3;
%產生空的dat表格
dat = table;
for gno = 1:gtotal
    fname = sprintf('Navon_G%d.csv',gno);
    % 讀入資料檔dat0
    dat0 = readtable(fname);
    %產生空的帶有group欄位的表格dat0g
    dat0g = table;
    %用gno填入dat0g.group
    dat0g.group = ones(height(dat0),1)*gno;
    % 將dat0併在dat0g的右邊
    dat0g = [dat0g dat0];
    % 將dat0g併在dat0的下面
    dat = [dat;dat0g];
end

% PART 2
% 資料排除-999, 其他為有效資料
vdat = dat(dat.respCorr ~= 3,:);
% 取得學號
uid = unique(vdat.id);
% 計算人數
totalN = length(uid);
%產生空的統計結果表格stats
stats = table;
for sno = 1:totalN
    for task = 1:2
        for cong = 1:2
            %挑出特定學號uid(sno)的特定task的特定cong
            vdat_cond = vdat(vdat.id == uid(sno) & vdat.task ==task & vdat.cong == cong,:);
            %把id,group,task,cong先填入該特定情境條件的結果表格stat0
            stats0 = vdat_cond(1,[1 2 5 7]); 
            %計算有效嘗試次
            stats0.ntrial = height(vdat_cond);
            %計算正確率：將正確code變成1，錯誤code變成0，再取平均值
            stats0.CR = mean(2-vdat_cond.respCorr);
            %挑選出排除極端值的rt
            goodRT = vdat_cond.rt(vdat_cond.respCorr==1 & vdat_cond.rt>=400 & vdat_cond.rt <= 800);
            %計算平均rt
            stats0.RT = mean(goodRT);
            %將單一情境的統計結果stats0合併在stats的下面
            stats =[stats;stats0];
        end
    end
end

% PART 3
%將stats統計結果存成csv檔
writetable(stats,'navon_mean.csv');
%建立空的總平均值矩陣
stats_final = zeros(4,6);
for task = 1:2
    for cong = 1:2
        %挑出個別情境的數值
        vstats = stats(stats.task == task & stats.cong == cong,:);
        %計算各變項平均值
        meantable = mean(vstats(:,{'ntrial','CR','RT'}));
        %將兩變項code組合成情境1-4
        cno = (task-1)*2+cong;
        %將結果寫入總平均值矩陣stats_final
        stats_final(cno,:) = [height(vstats) vstats.task(1) vstats.cong(1) table2array(meantable)];
    end
end

% 格式化顯示整體統計值
fprintf('N   task cong ntrials  crate    rt\n');
fprintf('%2d %4d %4d %7.1f %7.2f %7.2f\n',stats_final');
```

**重要概念**

1. 要取出table裡的資料用.，要挑出原本的欄位變成新表格用(:,n)

```matlab
id1 = dat(:,1)%table
id = dat.id(1)%double
```

1. 如果要取代資料名稱，兩邊資料型態要一樣

```matlab
dat.Properties.VariableNames(3) = {'block'}
dat.Properties.VariableNames{3} = 'block'
```

1. 取出受試者人數或特定數值個數用 unique

```matlab
totalN = length(unique(dat.id)）
```

### 作業4

![Untitled](%E6%9C%9F%E6%9C%AB%E8%80%83%20caceca0fc8b649ad8a2ca5b2cf66c00a/Untitled.png)

**程式碼**

```matlab
clearvars;
clc
resp_keynames = { 'a', 'l'};
sti = '鼠牛虎兔龍蛇馬羊猴雞狗豬';
total_trials = length(sti);
% 隨機項目編號的順序
itemno = Shuffle([1:total_trials]);
% 清空既存的按鍵
while(KbCheck(-1))
end
fprintf('實驗說明\n');
fprintf('按任意鍵開始\n');
%等待看完指導語後按鍵
KbWait(-1);
%產生空的table
data = table;
%指定data.tno, data.itemno, 內容需轉置成直欄
data.tno = [1:total_trials]';
% 作業未提到itemno,不計分
data.itemno = itemno';
for i = 1:total_trials
    citem = itemno(i);
    % 指定table第i嘗試次的rt & kno 為0
    data.rt(i) = 0;
    data.kno(i) = 0;
    % 清空既存的按鍵，再呈現十字
    while(KbCheck(-1))
    end
    clc;
    fprintf('＋\n');
    % 決定十字呈現的時間
    WaitSecs(.2+rand*.3);
    clc;
    %呈現刺激
    fprintf('%s\n',sti(citem));
    %計時開始
    t_start = GetSecs;
    t_end = t_start;
    % 需先指定kno為0， 否則超時記錄到的kno為前一嘗試的kno
    kno = 0;
    % 超過三秒就中止檢查按鍵的迴圈
    while (t_end-t_start <=3)
        [~,~,keycode] = KbCheck(-1);
        % 取得當下時間點
        t_end = GetSecs;
        % 若有按鍵，取得對應的按鍵名稱
        if (any(keycode))
            kstr = KbName(keycode);
            % 判斷按鍵名稱是否在keynames中，a->1 , l -> 2, 其他 -> 0
            [~, kno] = ismember(kstr,resp_keynames);
            % 若按鍵為a或l, 中斷迴圈
            if kno ~= 0
                break;
            end
        end
    end
    % 計算並記錄反應時間和按鍵
    data.kno(i) = kno;
    data.rt(i) = t_end-t_start;
end
```

**重要概念**

- 隨機：Shuffle
    - 一般的寫法：rand、randperm
- 暫停：WaitSecs(s)
    - 一般的寫法： 取代 pause(s)
- 指導語繼續鍵：KbWait
    - [~,keycode] = KbWait;
        - if keycode ~= 0 （如果有按鍵）
    - KbWait;
- 計時：GetSecs
    - 一般的寫法：tic、toc
- 鍵盤KbCheck
    - [keyIsDown, ~, keyCode] = KbCheck;
    - if keyIsDown
    - if keyCode(KbName('a'))
    - kstr = KbName(keycode);
- 滑鼠
    - [clicks, x, y] = GetClicks
    - [x,y,buttons] = GetMouse

### 作業5

<aside>
<img src="https://www.notion.so/icons/defibrillator_gray.svg" alt="https://www.notion.so/icons/defibrillator_gray.svg" width="40px" /> 奇偶判斷作業

</aside>

- 實驗程式進入Screen模式後，呈現一頁實驗指導語，描述實驗程序和必要說明，內容自訂
- 在每個嘗試次進行時，先出現凝視十字，呈現時間為介於300至400毫秒的隨機值
- 凝視十字消失後，隨機出現介於1至12的數字，每個數字在整個實驗均出現二次
- 左shift 和 右shift 為受試者反應鍵，分別對應為奇數反應鍵和偶數反應鍵
- 每個嘗試次根據反應時間快慢，呈現回饋訊息：以300毫秒和600毫秒做為臨界值，依反應時間長短呈現三種不同訊息

**程式碼**

```matlab
clearvars;
clc;

resp_keynames = {'LeftShift', 'RightShift'};
[w0, rect] = Screen('OpenWindow',0,0,[0 0 1024 768]);

%刺激材料:兩直條去隨機再拉成一長條，確保前12個各出現一次
stis=repmat([1:12]',1,2);
stis=Shuffle(stis);
stis=stis(:);
total_trials= length(stis);

% 清空既存的按鍵
while(KbCheck(-1))
end

%讀檔案:製作sti feedback,fix的textture（其他直接在w0 putimage）

sti = Screen('MakeTexture',w0,zeros(768, 1024));
feedback = Screen('MakeTexture',w0,zeros(768, 1024)); 

img = imread('fix.png'); %十字凝視texture
fix= Screen('MakeTexture',w0,img);

%產生空的table
data=table;
data.tno = [1:total_trials]';
data.itemno =stis;

%實驗開始
img = imread('instro.png'); %呈現指導語
Screen('PutImage',w0,img)
Screen('Flip',w0);
KbWait;

img = imread('start.png'); %呈現開始
Screen('PutImage',w0,img)
Screen('Flip',w0);
% 清空既存的按鍵
while(KbCheck(-1))
end
KbWait;  

for i = 1:total_trials
       citem = stis(i);  
      % 指定table第i嘗試次的rt & kno 為0
       data.rt(i) = 0;
       data.kno(i) = 0;
      % 清空既存的按鍵，再呈現十字
       while(KbCheck(-1))
       end

       %根據刺激材料準備圖片
       stinum = sprintf('p%d.png',stis(i));
       img = imread(stinum);
       Screen('PutImage',sti,img); 

       Screen('DrawTexture',w0,fix); %呈現十字凝視
       Screen('Flip',w0);
       WaitSecs(.3+rand*.1); %決定十字呈現的時間

       Screen('DrawTexture',w0,sti); %呈現刺激
       Screen('Flip',w0);

       %計時開始
       t_start = GetSecs;
       t_end = t_start;
       % 需先指定kno為0， 否則超時記錄到的kno為前一嘗試的kno
       kno = 0;
       data.kno(i) = kno;
       data.rt(i) = -999;
       % 超過三秒就中止檢查按鍵的迴圈
        while (t_end-t_start <=3)
            [~,~,keycode] = KbCheck(-1);
            % 取得當下時間點
            t_end = GetSecs;
            % 若有按鍵，取得對應的按鍵名稱
            if (any(keycode))
               kstr = KbName(keycode);
               % 判斷按鍵名稱是否在keynames中，LeftShift->1 ,RightShift -> 2, 其他 -> 0
               [~, kno] = ismember(kstr,resp_keynames);
               % 若按鍵為LeftShift,RightShift,呈現回饋並中斷迴圈
               if kno ~= 0
                    % 計算並記錄反應時間和按鍵
                    data.kno(i) = kno;
                    data.rt(i) = t_end-t_start;
                    %根據反應時間給予回饋（三種情況）
                    if t_end-t_start >.6
                        img = imread('600ms.png');
                    elseif  t_end-t_start  <.3
                        img = imread('300ms.png');                
                    else
                        img = imread('500ms.png');
                    end
                    break;
                end
            end
        end
    if(kno==0)
        img = imread('timeout.png');
    end
    Screen('PutImage',feedback,img); 
    Screen('DrawTexture',w0,feedback);
    Screen('Flip',w0);
    WaitSecs(0.4);
end
writetable(data,'result.csv');
img = imread('end.png');
Screen('PutImage',w0,img); %實驗結束
Screen('Flip',w0);
GetClicks;
Screen('CloseAll')
```

**重要概念**

- 螢幕設定
    - [w0, rect] = Screen('OpenWindow',0,0,[0 0 1024 768]);
    - 對角線座標(0,0)和（1024,768），顯示螢幕尺寸為1024*768
- 插入照片：
    - img = imread('檔名.png');
    - Screen('PutImage',w0,img)
    - Screen('Flip',w0);
- 清空既存的按鍵：
    - while(KbCheck(-1))
    end

## 其他（聲音與文字呈現）

### 文字呈現

Screen('TextFont',w0,'Kaiti TC');

Screen('TextSize',w0,200);
Screen('TextStyle',w0,0);

Screen('DrawText',w0,double('測試'),rect(3)/2,rect(4)/2,255);

Screen('Flip',w0);

### 聲音

[y, fs] = audioread(filename)

sound(y.fs);
	

****
	

****