---
title: "视频编码大作业"
published: 2025-01-01
description: "好的，这是一个《视频编码标准及其算法原理》的大作业。我会针对你提出的每一项任务要求，提供详细的步骤、原理、公式，并指出在MATLAB中可能用到的函数或方法。"
tags:
  - "视频编码"
  - "MATLAB"
category: "杂物"
categoryPath:
  - "杂物"
draft: false
---

好的，这是一个《视频编码标准及其算法原理》的大作业。我会针对你提出的每一项任务要求，提供详细的步骤、原理、公式，并指出在MATLAB中可能用到的函数或方法。

**请注意：** 由于我无法直接访问你的图像帧F1和F2，也无法执行MATLAB代码，因此我将提供的是一个框架和原理性的指导。你需要准备好你的图像数据，并根据这些指导在MATLAB中实现具体的代码。

---

**1. 任务要求：使用matlab完成所有流程**

这表示所有后续步骤的计算、变换、编码等都需要你通过编写MATLAB脚本来实现。

---

**2. 对帧F1进行JPEG编码（帧内编码）** 利用matlab实现JPEG的编码和解码重建第一帧F1’，并计算重建图像帧和原始帧F1之间的峰值信噪比PSNR。

<!--more-->

**JPEG编码流程概述：**

1. **颜色空间转换（可选）：** 如果是彩色图像，通常从RGB转换到YCbCr。对亮度(Y)和色度(Cb, Cr)分量分别处理。为了简化，我们假设F1是灰度图，或者我们只处理亮度分量。
2. **分块：** 将图像分成8x8的像素块。
3. **离散余弦变换 (DCT)：** 对每个8x8块进行2D-DCT。
4. **量化：** 对DCT系数进行量化，使用标准的量化表。
5. **Zigzag扫描：** 将量化后的2D系数矩阵转换为1D序列。
6. **熵编码：**
    - **DC系数编码：** 对DC系数（每个块的第一个系数，代表平均亮度）进行差分脉冲编码调制 (DPCM)，然后对差值进行熵编码（通常是霍夫曼编码）。
    - **AC系数编码：** 对AC系数（块内其余63个系数）进行游程编码 (RLE)，然后对(run, level)对进行熵编码（通常是霍夫曼编码）。
7. **码流组合：** 将编码后的数据组合成最终的JPEG码流。

**JPEG解码流程概述：**

1. **熵解码：** 解码DC和AC系数。
2. **反Zigzag扫描：** 将1D序列恢复成8x8的量化系数矩阵。
3. **反量化：** 对量化系数进行反量化。
4. **反离散余弦变换 (IDCT)：** 对每个块进行2D-IDCT。
5. **图像重建：** 将处理后的8x8块拼接成重建图像。
6. **颜色空间反转换（可选）：** 如果初始进行了转换，则从YCbCr转回RGB。

**MATLAB 实现步骤 (以灰度图为例):**

- **读取图像F1：**
  
    Matlab
    
    ```
    F1 = imread('your_frame_F1.png'); % 假设是png格式，替换为你的文件名
    if size(F1, 3) == 3
        F1_gray = rgb2gray(F1); % 如果是彩色图，转为灰度图
    else
        F1_gray = F1;
    end
    F1_double = double(F1_gray); % 转换为double类型方便计算
    [rows, cols] = size(F1_double);
    ```
    
- **JPEG编码 (简化版，主要关注DCT、量化、反量化、IDCT):**
  
    - **定义8x8块处理函数 (编码):**
      
        Matlab
        
        ```
        % 标准亮度量化表 (来自JPEG K.1)
        Q_lum = [16 11 10 16 24 40 51 61;
                 12 12 14 19 26 58 60 55;
                 14 13 16 24 40 57 69 56;
                 14 17 22 29 51 87 80 62;
                 18 22 37 56 68 109 103 77;
                 24 35 55 64 81 104 113 92;
                 49 64 78 87 103 121 120 101;
                 72 92 95 98 112 100 103 99];
        
        dct_func = @(block_struct) dct2(block_struct.data - 128); % DCT，减128使像素值中心化
        quant_func = @(block_struct) round(block_struct.data ./ Q_lum); % 量化
        
        % 对F1_double进行分块DCT和量化
        F1_dct_quant = blockproc(F1_double, [8 8], dct_func);
        F1_quantized = blockproc(F1_dct_quant, [8 8], quant_func);
        ```
        
    - **JPEG解码 (简化版):**
      
        - **定义8x8块处理函数 (解码):**
        
        Matlab
        
        ```
        dequant_func = @(block_struct) block_struct.data .* Q_lum; % 反量化
        idct_func = @(block_struct) idct2(block_struct.data) + 128; % IDCT，加128恢复
        
        % 对量化系数进行反量化和IDCT
        F1_dequant = blockproc(F1_quantized, [8 8], dequant_func);
        F1_reconstructed_double = blockproc(F1_dequant, [8 8], idct_func);
        F1_prime = uint8(F1_reconstructed_double); % 转回uint8
        ```
        
    - **注意：** 上述简化版省略了熵编码部分，因为完整的熵编码比较复杂。对于作业，你可能需要更详细地实现Zigzag扫描和基于霍夫曼表的编码，或者根据老师要求明确是否需要完整熵编码。
- **计算PSNR：**
  
    Matlab
    
    ```
    mse = mean((F1_double(:) - F1_reconstructed_double(:)).^2);
    if mse == 0
        psnr_val = Inf;
    else
        max_pixel_val = 255; % 假设8位图像
        psnr_val = 10 * log10(max_pixel_val^2 / mse);
    end
    fprintf('PSNR between F1 and F1_prime: %.2f dB\n', psnr_val);
    
    % 显示图像 (可选)
    % figure;
    % subplot(1,2,1); imshow(F1_gray); title('Original F1');
    % subplot(1,2,2); imshow(F1_prime); title('Reconstructed F1''');
    ```
    

---

**3. 从F1中任选一个8*8的块B1，给出以下详细过程**

假设你已经从`F1_double`中选择了一个8x8的块 `B1`。

Matlab

```
% 假设你选择的是F1左上角的第一个8x8块
B1 = F1_double(1:8, 1:8);
```

**3.1 给出该块的8*8矩阵形式：**

Matlab

```
disp('3.1 Block B1 (8x8 matrix):');
disp(B1);
```

这里会直接打印出你选定的`B1`矩阵。

**3.2 对块B1进行二维DCT变换，可以借助matlab计算得到。给出变换公式和变换系数块：**

- **二维DCT变换公式：** 对于一个 N×N 的块 f(x,y)，其2D-DCT系数 F(u,v) 计算如下： F(u,v)=C(u)C(v)x=0∑N−1​y=0∑N−1​f(x,y)cos[2N(2x+1)uπ​]cos[2N(2y+1)vπ​] 其中 u,v=0,1,…,N−1，且 C(k)={1/N![](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="400em" height="1.28em" viewBox="0 0 400000 1296" preserveAspectRatio="xMinYMin slice"><path d="M263,681c0.7,0,18,39.7,52,119
    c34,79.3,68.167,158.7,102.5,238c34.3,79.3,51.8,119.3,52.5,120
    c340,-704.7,510.7,-1060.3,512,-1067
    l0 -0
    c4.7,-7.3,11,-11,19,-11
    H40000v40H1012.3
    s-271.3,567,-271.3,567c-38.7,80.7,-84,175,-136,283c-52,108,-89.167,185.3,-111.5,232
    c-22.3,46.7,-33.8,70.3,-34.5,71c-4.7,4.7,-12.3,7,-23,7s-12,-1,-12,-1
    s-109,-253,-109,-253c-72.7,-168,-109.3,-252,-110,-252c-10.7,8,-22,16.7,-34,26
    c-22,17.3,-33.3,26,-34,26s-26,-26,-26,-26s76,-59,76,-59s76,-60,76,-60z
    M1001 80h400000v40h-400000z"></path></svg>)​2/N![](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="400em" height="1.28em" viewBox="0 0 400000 1296" preserveAspectRatio="xMinYMin slice"><path d="M263,681c0.7,0,18,39.7,52,119
    c34,79.3,68.167,158.7,102.5,238c34.3,79.3,51.8,119.3,52.5,120
    c340,-704.7,510.7,-1060.3,512,-1067
    l0 -0
    c4.7,-7.3,11,-11,19,-11
    H40000v40H1012.3
    s-271.3,567,-271.3,567c-38.7,80.7,-84,175,-136,283c-52,108,-89.167,185.3,-111.5,232
    c-22.3,46.7,-33.8,70.3,-34.5,71c-4.7,4.7,-12.3,7,-23,7s-12,-1,-12,-1
    s-109,-253,-109,-253c-72.7,-168,-109.3,-252,-110,-252c-10.7,8,-22,16.7,-34,26
    c-22,17.3,-33.3,26,-34,26s-26,-26,-26,-26s76,-59,76,-59s76,-60,76,-60z
    M1001 80h400000v40h-400000z"></path></svg>)​​if k=0if k>0​ 对于JPEG，通常先将像素值 f(x,y) 减去 2P−1 (例如，对于8位图像，P=8，减去128)。
    
- **MATLAB计算：**
  
    Matlab
    
    ```
    B1_centered = B1 - 128; % 中心化
    B1_dct = dct2(B1_centered);
    
    disp('3.2 DCT Transform Coefficients for B1:');
    disp(B1_dct);
    ```
    

**3.3 对变换系数块进行量化** 从jpeg标准文档Table K.1 – Luminance quantization table获取亮度分量的量化矩阵。给出量化过程的公式及量化后的系数矩阵：

- **亮度分量量化矩阵 (Table K.1)：** (已在上面给出)
  
    Matlab
    
    ```
    Q_lum = [16 11 10 16 24 40 51 61;
             12 12 14 19 26 58 60 55;
             14 13 16 24 40 57 69 56;
             14 17 22 29 51 87 80 62;
             18 22 37 56 68 109 103 77;
             24 35 55 64 81 104 113 92;
             49 64 78 87 103 121 120 101;
             72 92 95 98 112 100 103 99];
    ```
    
- **量化过程公式：** 对于DCT系数矩阵 F(u,v) 和量化矩阵 Q(u,v)，量化后的系数 Fq​(u,v) 为： Fq​(u,v)=round(Q(u,v)F(u,v)​)
  
- **MATLAB计算：**
  
    Matlab
    
    ```
    B1_quantized = round(B1_dct ./ Q_lum);
    
    disp('3.3 Quantized DCT Coefficients for B1:');
    disp(B1_quantized);
    ```
    

**3.4 对量化系数矩阵进行Z形扫描，得到一维序列:**

- **Z形扫描顺序：**
  
    ```
    0  1  5  6 14 15 27 28
    2  4  7 13 16 26 29 42
    3  8 12 17 25 30 41 43
    9 11 18 24 31 40 44 53
    10 19 23 32 39 45 52 54
    20 22 33 38 46 51 55 60
    21 34 37 47 50 56 59 61
    35 36 48 49 57 58 62 63
    ```
    
- **MATLAB实现 (需要一个辅助函数)：**
  
    Matlab
    
    ```
    % Zigzag scan order for 8x8 block
    zigzag_order = [
         0  1  5  6 14 15 27 28
         2  4  7 13 16 26 29 42
         3  8 12 17 25 30 41 43
         9 11 18 24 31 40 44 53
        10 19 23 32 39 45 52 54
        20 22 33 38 46 51 55 60
        21 34 37 47 50 56 59 61
        35 36 48 49 57 58 62 63
    ] + 1; % MATLAB is 1-indexed
    
    B1_zigzag_scanned = zeros(1, 64);
    temp_B1_quantized = B1_quantized'; % Transpose for column-major zigzag_order scan
    for i = 1:64
        B1_zigzag_scanned(i) = temp_B1_quantized(zigzag_order(i));
    end
    % A more robust way to implement zigzag:
    % (You can find many zigzag scan functions for MATLAB online)
    % For example:
    % function output = zigzag(input)
    %     [m, n] = size(input);
    %     output = zeros(1, m*n);
    %     count = 1;
    %     for s = 1:(m+n-1)
    %         if mod(s, 2) == 1 % Odd sum: up-right
    %             for i = max(1, s-n+1):min(s, m)
    %                 j = s - i + 1;
    %                 output(count) = input(i, j);
    %                 count = count + 1;
    %             end
    %         else % Even sum: down-left
    %             for j = max(1, s-m+1):min(s, n)
    %                 i = s - j + 1;
    %                 output(count) = input(i, j);
    %                 count = count + 1;
    %             end
    %         end
    %     end
    % end
    % B1_zigzag_scanned = zigzag(B1_quantized);
    
    
    disp('3.4 Zigzag Scanned 1D Sequence:');
    disp(B1_zigzag_scanned);
    ```
    

**3.5 对一维序列进行熵编码，包括给出中间符号1和中间符号2，最后给出该块的码流：** （可能需要用到JPEG标准文档...Table K.3 – Table for luminance DC coefficient differences，Table K.5 – Table for luminance AC coefficients。）

这部分是JPEG编码中最复杂的部分之一，涉及到差分编码、游程编码和霍夫曼编码。 你需要JPEG标准文档中定义的霍夫曼表（K.3, K.4, K.5, K.6）。通常这些表是固定的，或者在JPEG文件头中指定。

- **DC系数编码：**
  
    1. **差分编码 (DPCM):** 第一个块的DC系数直接编码。后续块的DC系数是当前块DC与前一个块DC的差值 (DIFFERENCE = DC_current - DC_previous)。 `DC_coeff = B1_zigzag_scanned(1);` `DIFFERENCE = DC_coeff - Previous_DC;` (假设`Previous_DC`是前一个块的DC，对第一个块，`Previous_DC = 0`)
    2. **确定类别 (Category/SSSS):** 根据DIFFERENCE的值，查Table K.3（亮度DC系数差值表）确定其类别 (SSSS)。
    3. **确定幅值 (Magnitude/ZZZZ...):** 对于非零DIFFERENCE，其二进制表示（如果为负，则为其绝对值的反码）是幅值。
    4. **霍夫曼编码：**
        - 类别的霍夫曼码 (from Table K.3, Luminance DC Huffman Code Table)。
        - 幅值的二进制码。
        - **中间符号1 (DC):** (Category, Magnitude_Code)
        - **码流 (DC):** HuffmanCode(Category) || Magnitude_Code
- **AC系数编码：**
  
    1. **游程编码 (RLE):** 从Zigzag序列的第二个系数开始，扫描AC系数。
        - 统计连续的0的个数 (RUNLENGTH/RRRR)。
        - 下一个非零系数的值 (LEVEL/VALUE)。
        - **特殊符号：**
            - EOB (End of Block): 如果剩下的AC系数都是0，则用一个特殊码 (通常是 (0,0) )表示。
            - ZRL (Zero Run Length): 如果有超过15个连续的0，则用一个特殊码 (通常是 (15,0) )表示16个0，然后继续计数。
    2. **确定 (RUNLENGTH, SIZE) 对：**
        - RUNLENGTH: 连续0的个数 (0-15)。
        - SIZE (or Category): 非零AC系数的幅值类别 (类似于DC系数的SSSS，但查AC系数的表)。
    3. **确定幅值 (Magnitude/ZZZZ...):** 非零AC系数的二进制表示。
    4. **霍夫MAN编码：**
        - (RUNLENGTH, SIZE)对的霍夫曼码 (from Table K.5, Luminance AC Huffman Code Table)。
        - 幅值的二进制码。
        - **中间符号2 (AC):** (Run, Size), Magnitude_Code
        - **码流 (AC):** HuffmanCode(Run, Size) || Magnitude_Code
- **示例 (概念性):** 假设 `B1_zigzag_scanned = [52, -5, 0, 3, -2, 0, 0, 1, EOB, ...]` (EOB代表后面都是0) 假设 `Previous_DC = 40`
  
    - **DC系数:** `DC_coeff = 52` `DIFFERENCE = 52 - 40 = 12` 查Table K.3: 12属于类别4 (SSSS=4)。幅值码是 '1100'。 假设类别4的霍夫曼码是 '100' (查表)。 DC码流: '100' + '1100' = '1001100'
      
    - **AC系数:**
      
        1. `-5`: RUNLENGTH=0. `-5` 属于类别3 (SIZE=3)。幅值码是 '010' (5的反码是...1010，取后3位)。 假设 (0,3) 的霍夫曼码是 '01' (查Table K.5)。 AC码流1: '01' + '010' = '01010'
        2. `0, 3`: RUNLENGTH=1. `3` 属于类别2 (SIZE=2). 幅值码是 '11'. 假设 (1,2) 的霍夫曼码是 '11010' (查Table K.5)。 AC码流2: '11010' + '11' = '1101011'
        3. `-2`: RUNLENGTH=0. `-2` 属于类别2 (SIZE=2). 幅值码是 '01'. 假设 (0,2) 的霍夫曼码是 '00' (查Table K.5)。 AC码流3: '00' + '01' = '0001'
        4. `0, 0, 1`: RUNLENGTH=2. `1` 属于类别1 (SIZE=1). 幅值码是 '1'. 假设 (2,1) 的霍夫曼码是 '11111001' (查Table K.5)。 AC码流4: '11111001' + '1' = '111110011'
        5. `EOB`: 假设 EOB (0,0) 的霍夫曼码是 '1010' (查Table K.5)。 AC码流_EOB: '1010'
    - **该块的总码流 (B1_bitstream):** DC码流 + AC码流1 + AC码流2 + ... + AC码流_EOB `B1_bitstream = '1001100' + '01010' + '1101011' + '0001' + '111110011' + '1010'`
      
    
    **MATLAB实现提示：** 你需要将JPEG标准中的霍夫曼表（K.3, K.5等）硬编码到你的MATLAB脚本中，通常是以cell数组或结构体的形式存储码字和对应的(类别)或(Run,Size)。然后根据计算出的DIFFERENCE、(RUNLENGTH, SIZE)查找对应的霍夫曼码。
    
    Matlab
    
    ```
    % --- DC Coefficient Encoding ---
    dc_coeff = B1_zigzag_scanned(1);
    % Assume previous_dc_coeff is available (for first block, it's 0)
    % For simplicity, let's assume previous_dc_coeff = 0 for this standalone block example.
    % In a real sequence, you'd track it.
    previous_dc_coeff = 0; % Placeholder
    diff_dc = dc_coeff - previous_dc_coeff;
    
    % [dc_category, dc_magnitude_code] = get_dc_category_and_magnitude(diff_dc); % User function
    % dc_huffman_code = lookup_dc_huffman_table(dc_category); % User function using Table K.3
    
    % For example:
    % if diff_dc == 0, category = 0, magnitude_code = ''
    % if diff_dc > 0, category = floor(log2(diff_dc)) + 1, magnitude_code = dec2bin(diff_dc)
    % if diff_dc < 0, category = floor(log2(abs(diff_dc))) + 1, magnitude_code = dec2bin(bitcmp(uint16(abs(diff_dc)), category)) % Be careful with negative num representation
    
    % disp(['DC diff: ', num2str(diff_dc)]);
    % disp(['DC Category (SSSS): ', num2str(dc_category)]); % Intermediate Symbol 1 (part 1)
    % disp(['DC Magnitude Code: ', dc_magnitude_code]);     % Intermediate Symbol 1 (part 2)
    % disp(['DC Huffman Code: ', dc_huffman_code]);
    % block_bitstream = dc_huffman_code;
    % block_bitstream = [block_bitstream, dc_magnitude_code];
    
    % --- AC Coefficient Encoding ---
    ac_coeffs = B1_zigzag_scanned(2:end);
    run_length = 0;
    % ac_bitstream_part = '';
    % for i = 1:length(ac_coeffs)
    %     if ac_coeffs(i) == 0
    %         run_length = run_length + 1;
    %         if run_length == 16 % ZRL
    %             % [ac_huff_code_zrl] = lookup_ac_huffman_table(15, 0); % (15,0) for ZRL from Table K.5
    %             % ac_bitstream_part = [ac_bitstream_part, ac_huff_code_zrl];
    %             run_length = 0;
    %         end
    %     else % Non-zero AC coefficient
    %         level = ac_coeffs(i);
    %         % [ac_size, ac_magnitude_code] = get_ac_size_and_magnitude(level); % User function
    %         % [ac_huff_code] = lookup_ac_huffman_table(run_length, ac_size); % User function from Table K.5
    %
    %         % disp(['AC (Run, Size): (', num2str(run_length), ',', num2str(ac_size), ')']); % Intermediate Symbol 2 (part 1)
    %         % disp(['AC Magnitude Code: ', ac_magnitude_code]); % Intermediate Symbol 2 (part 2)
    %         % disp(['AC Huffman Code for (Run,Size): ', ac_huff_code]);
    %
    %         % ac_bitstream_part = [ac_bitstream_part, ac_huff_code, ac_magnitude_code];
    %         run_length = 0;
    %     end
    % end
    % If last coefficient was zero or loop finishes, add EOB
    % [ac_huff_code_eob] = lookup_ac_huffman_table(0, 0); % (0,0) for EOB from Table K.5
    % ac_bitstream_part = [ac_bitstream_part, ac_huff_code_eob];
    % block_bitstream = [block_bitstream, ac_bitstream_part];
    
    disp('3.5 Entropy Coding:');
    disp('Due to complexity, a full Huffman coding implementation is omitted here.');
    disp('You need to implement Huffman table lookups for DC (Table K.3) and AC (Table K.5).');
    disp('Intermediate Symbol 1 (DC): (Category, Magnitude_Code)');
    disp('Intermediate Symbol 2 (AC): (Run, Size), Magnitude_Code');
    disp('Final Bitstream for B1: Concatenation of Huffman codes and magnitude codes.');
    % disp(['Example B1 bitstream (conceptual): ', block_bitstream]);
    ```
    
    **重要提示：** 实现完整的JPEG熵编码（特别是查表和构造码流）是相当细致的工作。你需要非常仔细地参照JPEG标准文档中的附录K。
    

---

**4. 对F2进行帧间编码（以第一帧的重建帧F1'为参考帧）**

**4.1 运动估计** 在F2中自选一个4*4的块B2，利用全搜索算法进行运动估计，全搜索的matlab代码可在理工智课下载。给出各个搜索点对应的运动矢量MV及其绝对误差和SAE，并确定出最优的运动矢量：

- **准备数据：**
  
    Matlab
    
    ```
    % 假设F2已读取并转为灰度double
    % F2 = imread('your_frame_F2.png');
    % F2_gray = rgb2gray(F2);
    % F2_double = double(F2_gray);
    
    % F1_prime_double = F1_reconstructed_double; % 来自第2步的重建帧
    
    % 选择F2中的一个4x4块 B2
    % 例如，选择F2中 (r_start, c_start) 开始的4x4块
    r_start_B2 = 10; c_start_B2 = 10; % 示例坐标
    B2 = F2_double(r_start_B2 : r_start_B2+3, c_start_B2 : c_start_B2+3);
    disp('4.1 Motion Estimation for B2:');
    disp('Selected B2 from F2:');
    disp(B2);
    ```
    
- **全搜索算法 (Full Search Algorithm):**
  
    1. **定义搜索窗口：** 在参考帧 `F1_prime_double` 中，以块 `B2` 在 `F2` 中的相同位置为中心，定义一个搜索范围 (e.g., +/-7 像素水平和垂直，即15x15的搜索区域)。
    2. **遍历搜索点：** 在搜索窗口内，逐个像素移动参考块 (与`B2`同样大小，4x4)。
    3. **计算匹配准则：** 对于每一个搜索到的候选块，与 `B2` 计算SAD (Sum of Absolute Differences) 或 SAE (Sum of Absolute Errors)。 SAE(dx,dy)=∑i=0N−1​∑j=0M−1​∣B2(i,j)−F1ref′​(i+dx,j+dy)∣ 其中 (dx, dy) 是运动矢量，N=4, M=4。 F1ref′​(i+dx,j+dy) 是参考帧中对应位置的块。
    4. **确定最优运动矢量：** 具有最小SAE的(dx, dy)即为最优运动矢量。
- **MATLAB 实现 (概念，依赖 "理工智课" 的代码或自己实现):**
  
    Matlab
    
    ```
    search_range = 7; % 例如 +/-7 像素搜索范围
    min_sae = inf;
    best_mv = [0, 0];
    [rows_f1, cols_f1] = size(F1_prime_double);
    
    disp('Search Points, MVs, and SAEs:');
    
    % B2的中心在F1'中的对应位置
    center_r_f1 = r_start_B2;
    center_c_f1 = c_start_B2;
    
    for dy = -search_range : search_range
        for dx = -search_range : search_range
            % 当前搜索的参考块的左上角坐标
            ref_r_start = center_r_f1 + dy;
            ref_c_start = center_c_f1 + dx;
    
            % 检查边界
            if (ref_r_start >= 1 && ref_r_start+3 <= rows_f1 && ...
                ref_c_start >= 1 && ref_c_start+3 <= cols_f1)
    
                ref_block = F1_prime_double(ref_r_start : ref_r_start+3, ref_c_start : ref_c_start+3);
                current_sae = sum(abs(B2(:) - ref_block(:)));
    
                fprintf('MV = (%d, %d), SAE = %f\n', dx, dy, current_sae);
    
                if current_sae < min_sae
                    min_sae = current_sae;
                    best_mv = [dx, dy]; % 通常MV定义为 (当前帧位置 - 参考帧位置)
                                      % 或者 (参考帧位置 - 当前帧位置)
                                      % 这里 dx, dy 是参考帧相对于当前帧块的偏移
                                      % 如果MV = Pred_pos - Curr_pos, 则 MV = [dx, dy]
                end
            end
        end
    end
    
    fprintf('Optimal Motion Vector (MV_x, MV_y) = (%d, %d) with SAE = %f\n', best_mv(1), best_mv(2), min_sae);
    ```
    

**4.2 利用运动矢量进行运动补偿后获得帧间预测残差矩阵E（4*4）：**

- **运动补偿：** 使用找到的最优运动矢量 `best_mv` 从参考帧 `F1_prime_double` 中提取预测块 `P`。
  
- **计算残差：** E=B2−P
  
- **MATLAB 计算：**
  
    Matlab
    
    ```
    predicted_block_r_start = r_start_B2 + best_mv(2); %  MV_y
    predicted_block_c_start = c_start_B2 + best_mv(1); %  MV_x
    
    % 确保预测块在F1'的边界内
    if (predicted_block_r_start >= 1 && predicted_block_r_start+3 <= rows_f1 && ...
        predicted_block_c_start >= 1 && predicted_block_c_start+3 <= cols_f1)
        P_B2 = F1_prime_double(predicted_block_r_start : predicted_block_r_start+3, ...
                               predicted_block_c_start : predicted_block_c_start+3);
    else
        % 如果超出边界，通常用边界像素填充或使用一个默认块
        % 为简化，这里假设总在边界内，实际编码器需要处理边界情况
        P_B2 = zeros(4,4); % 或者其他填充策略
        disp('Warning: Predicted block for MV is out of F1'' bounds. Using zero block for P_B2.');
    end
    
    E_B2 = B2 - P_B2; % 残差矩阵
    
    disp('4.2 Inter-prediction Residual Matrix E (4x4):');
    disp(E_B2);
    ```
    

**4.3 对于残差矩阵E进行H.264标准的整数DCT变换和量化，给出逐步计算的结果：**

- **H.264 4x4 整数变换 (Integer DCT-like transform):** H.264 使用的是一种整数变换，它是DCT的近似，但只使用整数运算。 变换公式: Y=Cf​XCfT​ 其中 X 是输入的4x4残差块 EB2​， Cf​ 是变换矩阵： Cf​=![](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.875em" height="4.800em" viewBox="0 0 875 4800"><path d="M863,9c0,-2,-2,-5,-6,-9c0,0,-17,0,-17,0c-12.7,0,-19.3,0.3,-20,1
    c-5.3,5.3,-10.3,11,-15,17c-242.7,294.7,-395.3,682,-458,1162c-21.3,163.3,-33.3,349,
    -36,557 l0,1284c0.2,6,0,26,0,60c2,159.3,10,310.7,24,454c53.3,528,210,
    949.7,470,1265c4.7,6,9.7,11.7,15,17c0.7,0.7,7,1,19,1c0,0,18,0,18,0c4,-4,6,-7,6,-9
    c0,-2.7,-3.3,-8.7,-10,-18c-135.3,-192.7,-235.5,-414.3,-300.5,-665c-65,-250.7,-102.5,
    -544.7,-112.5,-882c-2,-104,-3,-167,-3,-189
    l0,-1292c0,-162.7,5.7,-314,17,-454c20.7,-272,63.7,-513,129,-723c65.3,
    -210,155.3,-396.3,270,-559c6.7,-9.3,10,-15.3,10,-18z"></path></svg>)​1211​11−1−2​1−1−12​1−21−1​![](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.875em" height="4.800em" viewBox="0 0 875 4800"><path d="M76,0c-16.7,0,-25,3,-25,9c0,2,2,6.3,6,13c21.3,28.7,42.3,60.3,
    63,95c96.7,156.7,172.8,332.5,228.5,527.5c55.7,195,92.8,416.5,111.5,664.5
    c11.3,139.3,17,290.7,17,454c0,28,1.7,43,3.3,45l0,1209
    c-3,4,-3.3,16.7,-3.3,38c0,162,-5.7,313.7,-17,455c-18.7,248,-55.8,469.3,-111.5,664
    c-55.7,194.7,-131.8,370.3,-228.5,527c-20.7,34.7,-41.7,66.3,-63,95c-2,3.3,-4,7,-6,11
    c0,7.3,5.7,11,17,11c0,0,11,0,11,0c9.3,0,14.3,-0.3,15,-1c5.3,-5.3,10.3,-11,15,-17
    c242.7,-294.7,395.3,-681.7,458,-1161c21.3,-164.7,33.3,-350.7,36,-558
    l0,-1344c-2,-159.3,-10,-310.7,-24,-454c-53.3,-528,-210,-949.7,
    -470,-1265c-4.7,-6,-9.7,-11.7,-15,-17c-0.7,-0.7,-6.7,-1,-18,-1z"></path></svg>)​ 变换后的系数矩阵 Y 中的元素需要进行后续的缩放（这是H.264变换与量化紧密结合的一部分）。
    
    一个完整的变换与缩放过程可以表示为 W=(Cf​XCfT​)⊗Ef​，其中 ⊗ 是元素 Hadamard 乘积， Ef​ 是一个缩放矩阵： Ef​=![](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.875em" height="4.800em" viewBox="0 0 875 4800"><path d="M863,9c0,-2,-2,-5,-6,-9c0,0,-17,0,-17,0c-12.7,0,-19.3,0.3,-20,1
    c-5.3,5.3,-10.3,11,-15,17c-242.7,294.7,-395.3,682,-458,1162c-21.3,163.3,-33.3,349,
    -36,557 l0,1284c0.2,6,0,26,0,60c2,159.3,10,310.7,24,454c53.3,528,210,
    949.7,470,1265c4.7,6,9.7,11.7,15,17c0.7,0.7,7,1,19,1c0,0,18,0,18,0c4,-4,6,-7,6,-9
    c0,-2.7,-3.3,-8.7,-10,-18c-135.3,-192.7,-235.5,-414.3,-300.5,-665c-65,-250.7,-102.5,
    -544.7,-112.5,-882c-2,-104,-3,-167,-3,-189
    l0,-1292c0,-162.7,5.7,-314,17,-454c20.7,-272,63.7,-513,129,-723c65.3,
    -210,155.3,-396.3,270,-559c6.7,-9.3,10,-15.3,10,-18z"></path></svg>)​a2ab/2a2ab/2​ab/2b2/4ab/2b2/4​a2ab/2a2ab/2​ab/2b2/4ab/2b2/4​![](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.875em" height="4.800em" viewBox="0 0 875 4800"><path d="M76,0c-16.7,0,-25,3,-25,9c0,2,2,6.3,6,13c21.3,28.7,42.3,60.3,
    63,95c96.7,156.7,172.8,332.5,228.5,527.5c55.7,195,92.8,416.5,111.5,664.5
    c11.3,139.3,17,290.7,17,454c0,28,1.7,43,3.3,45l0,1209
    c-3,4,-3.3,16.7,-3.3,38c0,162,-5.7,313.7,-17,455c-18.7,248,-55.8,469.3,-111.5,664
    c-55.7,194.7,-131.8,370.3,-228.5,527c-20.7,34.7,-41.7,66.3,-63,95c-2,3.3,-4,7,-6,11
    c0,7.3,5.7,11,17,11c0,0,11,0,11,0c9.3,0,14.3,-0.3,15,-1c5.3,-5.3,10.3,-11,15,-17
    c242.7,-294.7,395.3,-681.7,458,-1161c21.3,-164.7,33.3,-350.7,36,-558
    l0,-1344c-2,-159.3,-10,-310.7,-24,-454c-53.3,-528,-210,-949.7,
    -470,-1265c-4.7,-6,-9.7,-11.7,-15,-17c-0.7,-0.7,-6.7,-1,-18,-1z"></path></svg>)​ 其中 a=1/2, b=1/2![](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="400em" height="1.28em" viewBox="0 0 400000 1296" preserveAspectRatio="xMinYMin slice"><path d="M263,681c0.7,0,18,39.7,52,119
    c34,79.3,68.167,158.7,102.5,238c34.3,79.3,51.8,119.3,52.5,120
    c340,-704.7,510.7,-1060.3,512,-1067
    l0 -0
    c4.7,-7.3,11,-11,19,-11
    H40000v40H1012.3
    s-271.3,567,-271.3,567c-38.7,80.7,-84,175,-136,283c-52,108,-89.167,185.3,-111.5,232
    c-22.3,46.7,-33.8,70.3,-34.5,71c-4.7,4.7,-12.3,7,-23,7s-12,-1,-12,-1
    s-109,-253,-109,-253c-72.7,-168,-109.3,-252,-110,-252c-10.7,8,-22,16.7,-34,26
    c-22,17.3,-33.3,26,-34,26s-26,-26,-26,-26s76,-59,76,-59s76,-60,76,-60z
    M1001 80h400000v40h-400000z"></path></svg>)​≈0.707 (实际H.264中这些因子被整合到量化步骤的乘法因子和移位中，避免浮点运算)。
    
    **核心变换 (Forward transform core coefficients):** Yij​=∑k​∑l​Cik​Xkl​(CT)lj​ 实际上，H.264标准定义的是整数运算后的系数，然后通过量化参数QP进行量化。
    
    **简化版整数变换（只考虑核心变换 Y=Cf​XCfT​）：**
    
    Matlab
    
    ```
    Cf = [ 1  1  1  1;
           2  1 -1 -2;
           1 -1 -1  1;
           1 -2  2 -1 ];
    
    Y_B2 = Cf * E_B2 * Cf'; % 核心变换
    disp('4.3.1 H.264 Integer Transformed Coefficients (before scaling/quant):');
    disp(Y_B2);
    ```
    
- **H.264 量化:** 量化公式 (简化形式，实际更复杂，涉及QP和预计算的乘法因子和移位): Zij​=round(Yij​/Qstep​) 或者更接近H.264标准的形式（对于帧内预测的亮度残差系数，这里是帧间）： Levelij​=(Wij​⋅MFij​+f)>>(qbits+shift) 其中 Wij​ 是变换系数，MFij​ 是量化参数QP相关的乘法因子，f 是加性偏移（用于近似round），`>>` 是右移。 qbits=15+⌊QP/6⌋ MF 依赖于 QP(mod6) 和系数位置 (i,j)。
  
    **为简化演示，我们使用一个非常简化的量化，假设一个 Qstep:** 你需要选择一个量化参数QP。QP决定了 Qstep​。 例如，对于一个给定的QP，可以有一个对应的 Qstep​。 H.264的量化表和QP到 Qstep​ 的映射比JPEG复杂。 Qstep​ 大约每增加6个QP值就翻倍。
    
    **一个非常非常简化的例子（不完全符合H.264标准，仅为演示步骤）：** 假设我们选择一个QP，它对应一个 Qstep​。 例如，如果QP=26, Qstep​ 可能近似为 10 (这只是一个示意值)。
    
    Matlab
    
    ```
    QP = 26; % Example Quantization Parameter
    % In H.264, Qstep is derived from QP and coefficient position.
    % For this example, let's use a placeholder Qstep.
    % A rough Qstep might be derived, e.g. Qstep_base * 2^(QP/6).
    % Or use pre-defined scaling factors based on QP % 6 and QP / 6.
    
    % H.264 has 6 quantization scaling factors for each QP/6 level (V[][])
    % and a scaling factor based on QP%6 (PF[][])
    % LevelScale( qP, i, j ) = PF[qP%6][(i*4+j)% ( (i==0||i==2)&&(j==0||j==2) ? 3:1 ) ] << (qP/6)
    % QuantizedCoeff = sign(TransformedCoeff) * ( abs(TransformedCoeff) * V[qP%6][(i*4+j)%...] + offset ) >> ( 15 + qP/6 )
    
    % Simplified Quantization (conceptual for assignment unless deep dive required)
    % This does NOT reflect the full H.264 quantization.
    % The true H.264 quantization is:
    % Z_ij = sign(Y_ij) * (|Y_ij| * MF + f) >> (15 + floor(QP/6))
    % where MF depends on QP%6 and coefficient position (i,j) from a table,
    % f is an offset (e.g., 2^(14+floor(QP/6)) for intra, smaller for inter)
    
    % Let's create a placeholder quantization matrix based on QP for demonstration
    % This is NOT standard but illustrates the step.
    % The actual MF values are in tables like Table 7-9 in H.264 spec.
    % For simplicity, let's use a flat Qstep, which is a gross simplification.
    Qstep_example = 10 * (2^(QP/6 - 4)); % Very rough illustrative Qstep
    
    E_B2_quantized = round(Y_B2 / Qstep_example);
    
    disp(['4.3.2 H.264 Quantized Coefficients (using QP=', num2str(QP), ' and simplified Qstep=', num2str(Qstep_example), '):']);
    disp(E_B2_quantized);
    
    disp('NOTE: The H.264 quantization shown above is highly simplified.');
    disp('A proper implementation requires using H.264 standard tables for MF and f based on QP.');
    disp('The core integer transform part Y = Cf * E_B2 * Cf'' is correct.');
    disp('For accurate quantization, refer to H.264 standard section 7.4.5.');
    ```
    
    **逐步计算：**
    
    1. 原始残差块 EB2​
    2. 计算 Temp=EB2​⋅CfT​
    3. 计算 YB2​=Cf​⋅Temp (核心变换结果)
    4. 对 YB2​ 的每个系数，应用H.264量化公式（涉及QP、MF、移位等）。

**4.4 Zigzag形扫描与熵编码，根据H.264的熵编码顺序，给出逐步熵编码的产生的码流，给出该4*4块B2最终的的码流：（可能用到的表格为H.264标准文档中Table 9.5 ~9.10）**

- **4x4 Zigzag扫描：** H.264对4x4块的扫描通常也是Zigzag，但根据预测模式（如帧内4x4, 帧间等）可以有不同的扫描顺序。对于帧间残差，通常是标准的Zigzag。
  
    ```
    0  1  4  5
    2  3  6  7
    8  9 12 13
    10 11 14 15
    ```
    
    Matlab
    
    ```
    % 4x4 Zigzag scan order
    zigzag_order_4x4 = [
         0  1  4  5
         2  3  6  7
         8  9 12 13
        10 11 14 15
    ] + 1; % MATLAB 1-indexed
    
    B2_quant_zigzag = zeros(1, 16);
    temp_B2_quant = E_B2_quantized'; % Transpose for column-major scan
    for i = 1:16
        B2_quant_zigzag(i) = temp_B2_quant(zigzag_order_4x4(i));
    end
    disp('4.4.1 Zigzag scanned quantized coefficients for B2 residual:');
    disp(B2_quant_zigzag);
    ```
    
- **熵编码 (CAVLC - Context-Adaptive Variable Length Coding):** H.264对残差系数的熵编码（主要是CAVLC或CABAC）非常复杂。CAVLC是 baseline profile 的标准。 CAVLC编码一个块的非零系数的过程：
  
    1. **编码 `coeff_token`:**
       
        - 取决于 `TotalCoeffs` (块中非零系数的个数) 和 `TrailingOnes` (尾随的 +/-1 的个数，最多3个)。
        - `coeff_token` 从特定的VLC表 (Table 9-5, Table 9-6 in H.264) 中查找，基于 `TotalCoeffs` 和 `TrailingOnes`。上下文模型也可能影响表的选择 (e.g., nC: a measure of previously coded non-zero coefficients in neighboring blocks).
    2. **编码 `TrailingOnes` 的符号:** 每个 `TrailingOne` 的符号用1比特表示 (0 for +, 1 for -)。
       
    3. **编码剩下非零系数的 `level`:**
       
        - 从后往前（不包括 `TrailingOnes`）编码每个非零系数的 `level` (幅值)。
        - 使用 Golomb-Rice codes 或 Exp-Golomb codes (VLC tables)。
    4. **编码 `TotalZeros`:**
       
        - 编码在最后一个非零系数之前的所有0的总数 (`TotalZeros`)。
        - 使用VLC表 (Table 9-7)，基于 `TotalCoeffs`。
    5. **编码每个非零系数前的 `run_before` (0的游程):**
       
        - 从后往前，为每个非零系数（不包括最后一个）编码其前面有多少个0 (`run_before`)。
        - 使用VLC表 (Table 9-8, 9-9, 9-10)，取决于 `zerosLeft` (尚未编码的0的个数)。
        - 如果所有剩余的0都在第一个非零系数之前，则不需要再编码 `run_before`。
    
    **逐步熵编码的码流 (CAVLC 概念性示例):** 假设 `B2_quant_zigzag = [3, 0, -1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]`
    
    1. `TotalCoeffs = 3`
    2. `TrailingOnes`:
        - `1` is a trailing one.
        - `-1` is a trailing one.
        - `3` is not. So, `TrailingOnes = 2`.
    3. **`coeff_token`:** 查找Table 9-5 (假设nC < 2) for `TotalCoeffs=3`, `TrailingOnes=2`. 假设码字是 `'0000110'`
    4. **Signs of TrailingOnes:**
        - For `1`: sign is `+` (code `0`)
        - For `-1`: sign is `-` (code `1`) TrailingOnes signs bitstream: `'01'`
    5. **Levels of remaining coeffs (scan reverse, skip TrailingOnes):**
        - Remaining coeff is `3`.
        - Encode `level` = `3`. (Using Exp-Golomb or similar, e.g., `00100` for `3` if `level_prefix` and `level_suffix` rules are applied). Let's say `level_code_for_3 = '00111'` (this is an example, actual code depends on specific Exp-Golomb parameters/tables used by CAVLC for levels).
    6. **`TotalZeros`:**
        - Last non-zero coeff is `1` (at index 3 in zigzag). Before it, there is one `0` (at index 1).
        - So `TotalZeros = 1`.
        - Lookup Table 9-7 for `TotalCoeffs=3`, `TotalZeros=1`. Assume code is `'11'`.
    7. **`run_before` for each coeff (scan reverse, stop before last):**
        - Coeff `-1` (at index 2): `run_before` (zeros before it since previous non-zero) = `1` (the zero at index 1). `zerosLeft` is initially `TotalZeros = 1`. Lookup Table (e.g., 9-10) for `run_before=1` given `zerosLeft=1`. Assume code is `'1'`. `zerosLeft` becomes 0.
        - Coeff `3` (at index 0): This is the first non-zero coeff to be considered in this step (after trailing ones). No more runs to code if `zerosLeft` is 0 or it's the first significant coeff. (The logic for `run_before` is intricate; it's about runs of zeros _between_ the non-zero AC coefficients being coded in this phase).
    
    **最终码流 (B2_final_bitstream_CAVLC):** `'0000110' (coeff_token) + '01' (signs) + '00111' (level for 3) + '11' (TotalZeros) + '1' (run_before for -1)` `= '00001100100111111'`
    
    **MATLAB实现提示：** CAVLC的完整实现非常复杂，需要仔细处理上下文、查多个VLC表、以及管理系数的扫描顺序。
    
    Matlab
    
    ```
    disp('4.4.2 H.264 Entropy Coding (CAVLC):');
    disp('CAVLC is complex. The following is a conceptual outline:');
    disp('1. Calculate TotalCoeffs and TrailingOnes from B2_quant_zigzag.');
    disp('2. Encode coeff_token (using Table 9-5/9-6 based on nC, TotalCoeffs, TrailingOnes).');
    %   [coeff_token_bits] = get_coeff_token_cavlc(TotalCoeffs, TrailingOnes, nC_value);
    disp('3. Encode signs of TrailingOnes (1 bit each).');
    %   [trailing_ones_sign_bits] = get_trailing_ones_signs(B2_quant_zigzag, TrailingOnes);
    disp('4. Encode levels of remaining non-zero coefficients (Exp-Golomb).');
    %   [levels_bits] = encode_levels_cavlc(B2_quant_zigzag, TotalCoeffs, TrailingOnes);
    disp('5. Encode TotalZeros (zeros before last non-zero coeff, Table 9-7).');
    %   [total_zeros_bits] = encode_total_zeros_cavlc(B2_quant_zigzag, TotalCoeffs);
    disp('6. Encode run_before for zeros preceding each non-zero coeff (Tables 9-8/9-9/9-10).');
    %   [runs_bits] = encode_runs_cavlc(B2_quant_zigzag, TotalCoeffs, TotalZeros);
    % block_B2_final_stream = [coeff_token_bits, trailing_ones_sign_bits, levels_bits, total_zeros_bits, runs_bits];
    disp('The final bitstream for B2 is the concatenation of these parts.');
    disp('Actual bit values require implementing the lookup for H.264 Tables 9-5 to 9-10.');
    ```
    

---

这是一个相当全面的大作业。在MATLAB中实现这些，特别是JPEG的完整熵编码和H.264的CAVLC，需要花费大量时间和精力进行调试。建议你分模块实现和测试，并仔细阅读相关的标准文档。

祝你顺利完成作业！
