# 音频格式

**采样率**是指：声音信号在“模 —> 数”转换过程中单位时间内采样的次数，单位：kbps(k bit per second)。

**采样值**是指：每一次采样周期内声音模拟信号的积分值。

**单声道**的音频文件采样数据为8bit的短整数（short int）

**双声道**的音频文件采样数据为16bit的整数（int）,其中高八位为左声道，低八位为右声道。

## PCM

PCM(Pulse Code Modulation)也被称为 脉码编码调制。PCM中的声音数据没有被压缩，如果是单声道的文件，采样数据按时间的先后顺序依次存入。

PCM16LE的取值范围是（-32768~32767），PCM8的取值是（0~255） PCM16LE双声道数据16bit的采样位数转换为8bit，需要经过两个步骤：

1. 将（-32768~32767）转换为（-128~127）的8bit有符号值。
2. 将（-128~127）转换为（0~255）的8bit无符号值。

## WAVE

WAVE文件是一种RIFF格式的文件。其基本块名称是“WAVE”，其中包含了两个子块“fmt”和“data”。从编程的角度简单说来就是由 **WAVE\_HEADER、WAVE\_FMT、WAVE\_DATA、采样数据** 共4个部分组成。

WAVE格式是windows下最常见的音频格式（扩展名为.wav）,实质就是在PCM文件前加上了一个文件头。

```
typedef struct WAVE_HEADER{
    char fccID[4];
    unsigned long dwSize;
    char fccType[4];
}WAVE_HEADER;

typedef struct WAVE_FMT{
    char  fccID[4];
    unsigned long dwSize;
    unsigned short wFormatTag;
    unsigned short wChannels;
    unsigned long dwSamplesPerSec;
    unsigned long dwAvgBytesPerSec;
    unsigned short wBlockAlign;
    unsigned short uiBitsPerSample;
}WAVE_FMT;

typedef struct WAVE_DATA{
    char       fccID[4];
    unsigned long dwSize;
}WAVE_DATA; 
```



# 音频码流

## AAC(Advanced Audio Coding)高级音频编码

AAC原始码流（又称为“裸流”）是由一个一个的ADTS frame组成的。其中每个ADTS frame之间通过syncword（同步字）进行分隔，同步字为0xFFF（二进制“111111111111”）。AAC码流解析的步骤就是首先从码流中搜索0x0FFF，分离出ADTS frame；然后再分析ADTS frame的首部各个字段。

### ADTS（Audio Data Transport Stream）

ADTS的头部信息包含两部分

```
Header consists of 7 or 9 bytes(without or with CRC).（CRC = 循环冗余校验）。

bslbf（bit string, left bit first）： bit String, 左位在先.
uimsbf（unsigned integer，most significant bit first）： 无符号整数，高位在先

adts_fixed_header(){
    syncword;            //12bits(bslbf) all bits must be 1
    ID;                  //1bits(bslbf) MPEG Version: 0 for MPEG-4, 1 for MPEG-2
    layer;               //2bits(uimsbf)  always: '00'
    protection_absent;   //1bits(bslbf) Warning，0表示CRC，1表示非CRC
    profile;             //2bits(uimsbf) AAC的级别，main：0，low complexity：1，scalable sampling rate：2，reserved：3
    sampling_frequency_index; //4bits(uimsbf) 使用的采样率下标，通过这个下标在 Sampling Frequencies[]数组中查找得知采样率的值。
    private_bit;         //1bits(bslbf) 保证不会被MPEG使用 编码是设为0，解码时忽略
    channel_configuration;    //3bits(uimsbf) 声道配置
    original_copy;1 bslbf     //编码是设为0，解码时忽略
    home;1 bslbf        //编码是设为0，解码时忽略
}
adts_variable_header(){
    copyright_identiflaction_bit;   //1bits(bslbf) 下一个字节是版权标识符 编码是设为0，解码时忽略
    copyright_identification_start; //1bits(bslbf) 版权标识符的起始位  编码是设为0，解码时忽略
    acc_frame_length;               //13bits(bslbf) ADTS一帧的长度（包含头和原始流）,frameLenght = (protection_absent==1 ? 7 : 9) + size(AACFrame)
    adts_buffer_fullness;           //11bits(bslbf) ADTS
    number_of_raw_data_blocks_in_frame; //2bits(uimsfb)  AAC(RDB)帧数减一，为了最大的兼容性，每个ADTS帧始终使用1个AAC帧
}
```
