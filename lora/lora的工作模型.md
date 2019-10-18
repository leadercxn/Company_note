# lora的CAD模式

## SX1276：

> 概念 : 定时扫描信道，检测lora数据包的前导码，
降低功耗。随着扩频调制技术的应用，人们无法确认信道被占有，是由于低噪声影响，还是真的数据来到，所以引入了CAD检测。LoRa CAD检测方法：从机设置好频率和扩频因子，开启CAD模式，（注意：无论是否有信号到来，都会产生CADDone中断），当有匹配（相同的频率和扩频因子）的信号到来时，就会产生CADDetect中断，CADDone也会产生，并且，CADDetect和CADDone会同时产生。

[CAD模型的多节点检测介绍链接](http://www.pianshen.com/article/1026562256/;jsessionid=107825730CF41CCF2D201B4BFD374C09)<br>
[无线节点的空中唤醒技术](https://blog.csdn.net/iotisan/article/details/55695465)，介绍CAD的原理，以及 空中唤醒优化的方法 及 噪音的应对，传输锁相的介绍

> 数据帧的结构形式:<br>
![](https://raw.githubusercontent.com/leadercxn/SENSORO/a8d39bfa01b91ef11ab7d5d644835d63f6717bed/lora/lora数据帧的结构模型.png)
由于CAD检测数据包的前导码部分，因此要想实现空中唤醒，结合节点定期检测时间，需要设置合适的前导码发送时间，保证 前导码发送时间>节点定期检测时间 ，则需要设定一定的前导码长度，可通过配置RegPreambleMsb和RegPreambleLsb寄存器来实现。如下图所示，可将前导码寄存器长度设置在6-65536之间来改变发送前导码长度。

#### sx1276存在两种模式:
+ LORA模式
> LoRa调制解调器采用专利扩频调制和前向纠错技术，它融合了数字扩频、数字信号处理和前向纠错编码技术。<br>
![](https://raw.githubusercontent.com/leadercxn/SENSORO/8c77111c4b7c5a531438f15620284b0787944790/lora/lora扩频调制.png)

+ FSK模式
> 用两个频率承载二进制1和0的双频FSK系统<br>
![](https://raw.githubusercontent.com/leadercxn/SENSORO/8c77111c4b7c5a531438f15620284b0787944790/lora/FSK原理.png)

两者区别：<br>
![](https://raw.githubusercontent.com/leadercxn/SENSORO/8c77111c4b7c5a531438f15620284b0787944790/lora/lora和FSK的区别.png)



> 相关科普文：<br>
[lora跟lorawan的区别.pdf](https://wiki.ai-thinker.com/_media/lora/lorawan_faq问题.pdf)<br>
[1276/77/78 datasheet](https://www.zlg.cn/data/upload/software/Wireless/ZM470SX-M_SX1278-data-cn.pdf)<br>
[lora 技术pingpong系统](https://blog.csdn.net/weixin_39148042/article/details/81588897)<br>

> [SX127x芯片数字IO引脚映射](https://blog.csdn.net/HowieXue/article/details/78052758),此文含有大量学习链接。
SX1276/7/8的6个DIO通用IO引脚在LoRa模式下均可用。它们的映射关系取决于RegDioMapping1和RegDioMapping2这两个寄存器的配置，如下表：<br>
![](https://raw.githubusercontent.com/leadercxn/SENSORO/e0e061ea6324252dba57fffc49a4de02a8a4a31f/lora/LORA的DIO映射表.png)

-------------------------------------------------------------------------------------------------------





























































### 代码相关

+ sx1276的lora驱动接口
```C
struct Radio_s
{
      /*!
     * \brief Initializes the radio
     *
     * \param [IN] events Structure containing the driver callback functions
     */
//    void    ( *IoInit )( void );
  
        /*!
     * \brief Initializes the radio
     *
     * \param [IN] events Structure containing the driver callback functions
     */
//    void    ( *IoDeInit )( void );
    /*!
     * \brief Initializes the radio
     *
     * \param [IN] events Structure containing the driver callback functions
     * \param [OUT] returns radioWakeUpTime
     */
    uint32_t    ( *Init )( RadioEvents_t *events );
    /*!
     * Return current radio status
     *
     * \param status Radio status.[RF_IDLE, RF_RX_RUNNING, RF_TX_RUNNING]
     */
    RadioState_t ( *GetStatus )( void );
    /*!
     * \brief Configures the radio with the given modem
     *
     * \param [IN] modem Modem to be used [0: FSK, 1: LoRa] 
     */
    void    ( *SetModem )( RadioModems_t modem );
    /*!
     * \brief Sets the channel frequency
     *
     * \param [IN] freq         Channel RF frequency
     */
    void    ( *SetChannel )( uint32_t freq );
    /*!
     * \brief Sets the channels configuration
     *
     * \param [IN] modem      Radio modem to be used [0: FSK, 1: LoRa]
     * \param [IN] freq       Channel RF frequency
     * \param [IN] rssiThresh RSSI threshold
     *
     * \retval isFree         [true: Channel is free, false: Channel is not free]
     */
    bool    ( *IsChannelFree )( RadioModems_t modem, uint32_t freq, int16_t rssiThresh );
    /*!
     * \brief Generates a 32 bits random value based on the RSSI readings
     *
     * \remark This function sets the radio in LoRa modem mode and disables 
     *         all interrupts.
     *         After calling this function either Radio.SetRxConfig or
     *         Radio.SetTxConfig functions must be called.
     *
     * \retval randomValue    32 bits random value
     */
    uint32_t ( *Random )( void );
    /*!
     * \brief Sets the reception parameters
     *
     * \param [IN] modem        Radio modem to be used [0: FSK, 1: LoRa]
     * \param [IN] bandwidth    Sets the bandwidth
     *                          FSK : >= 2600 and <= 250000 Hz
     *                          LoRa: [0: 125 kHz, 1: 250 kHz,
     *                                 2: 500 kHz, 3: Reserved] 
     * \param [IN] datarate     Sets the Datarate
     *                          FSK : 600..300000 bits/s
     *                          LoRa: [6: 64, 7: 128, 8: 256, 9: 512,
     *                                10: 1024, 11: 2048, 12: 4096  chips]
     * \param [IN] coderate     Sets the coding rate (LoRa only)
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [1: 4/5, 2: 4/6, 3: 4/7, 4: 4/8] 
     * \param [IN] bandwidthAfc Sets the AFC Bandwidth (FSK only) 
     *                          FSK : >= 2600 and <= 250000 Hz
     *                          LoRa: N/A ( set to 0 ) 
     * \param [IN] preambleLen  Sets the Preamble length
     *                          FSK : Number of bytes 
     *                          LoRa: Length in symbols (the hardware adds 4 more symbols)
     * \param [IN] symbTimeout  Sets the RxSingle timeout value (LoRa only) 
     *                          FSK : N/A ( set to 0 ) 
     *                          LoRa: timeout in symbols
     * \param [IN] fixLen       Fixed length packets [0: variable, 1: fixed]
     * \param [IN] payloadLen   Sets payload length when fixed length is used
     * \param [IN] crcOn        Enables/Disables the CRC [0: OFF, 1: ON]
     * \param [IN] FreqHopOn    Enables disables the intra-packet frequency hopping
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [0: OFF, 1: ON]
     * \param [IN] HopPeriod    Number of symbols between each hop
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: Number of symbols
     * \param [IN] iqInverted   Inverts IQ signals (LoRa only)
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [0: not inverted, 1: inverted]
     * \param [IN] rxContinuous Sets the reception in continuous mode
     *                          [false: single mode, true: continuous mode]
     */
    void    ( *SetRxConfig )( RadioModems_t modem, uint32_t bandwidth,
                              uint32_t datarate, uint8_t coderate,
                              uint32_t bandwidthAfc, uint16_t preambleLen,
                              uint16_t symbTimeout, bool fixLen,
                              uint8_t payloadLen,
                              bool crcOn, bool FreqHopOn, uint8_t HopPeriod,
                              bool iqInverted, bool rxContinuous );
    /*!
     * \brief Sets the transmission parameters
     *
     * \param [IN] modem        Radio modem to be used [0: FSK, 1: LoRa] 
     * \param [IN] power        Sets the output power [dBm]
     * \param [IN] fdev         Sets the frequency deviation (FSK only)
     *                          FSK : [Hz]
     *                          LoRa: 0
     * \param [IN] bandwidth    Sets the bandwidth (LoRa only)
     *                          FSK : 0
     *                          LoRa: [0: 125 kHz, 1: 250 kHz,
     *                                 2: 500 kHz, 3: Reserved] 
     * \param [IN] datarate     Sets the Datarate
     *                          FSK : 600..300000 bits/s
     *                          LoRa: [6: 64, 7: 128, 8: 256, 9: 512,
     *                                10: 1024, 11: 2048, 12: 4096  chips]
     * \param [IN] coderate     Sets the coding rate (LoRa only)
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [1: 4/5, 2: 4/6, 3: 4/7, 4: 4/8] 
     * \param [IN] preambleLen  Sets the preamble length
     *                          FSK : Number of bytes 
     *                          LoRa: Length in symbols (the hardware adds 4 more symbols)
     * \param [IN] fixLen       Fixed length packets [0: variable, 1: fixed]
     * \param [IN] crcOn        Enables disables the CRC [0: OFF, 1: ON]
     * \param [IN] FreqHopOn    Enables disables the intra-packet frequency hopping
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [0: OFF, 1: ON]
     * \param [IN] HopPeriod    Number of symbols between each hop
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: Number of symbols
     * \param [IN] iqInverted   Inverts IQ signals (LoRa only)
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [0: not inverted, 1: inverted]
     * \param [IN] timeout      Transmission timeout [ms]
     */
    void    ( *SetTxConfig )( RadioModems_t modem, int8_t power, uint32_t fdev, 
                              uint32_t bandwidth, uint32_t datarate,
                              uint8_t coderate, uint16_t preambleLen,
                              bool fixLen, bool crcOn, bool FreqHopOn,
                              uint8_t HopPeriod, bool iqInverted, uint32_t timeout );
    /*!
     * \brief Checks if the given RF frequency is supported by the hardware
     *
     * \param [IN] frequency RF frequency to be checked
     * \retval isSupported [true: supported, false: unsupported]
     */
    bool    ( *CheckRfFrequency )( uint32_t frequency );
    /*!
     * \brief Computes the packet time on air in ms for the given payload
     *
     * \Remark Can only be called once SetRxConfig or SetTxConfig have been called
     *
     * \param [IN] modem      Radio modem to be used [0: FSK, 1: LoRa]
     * \param [IN] pktLen     Packet payload length
     *
     * \retval airTime        Computed airTime (ms) for the given packet payload length
     */
    uint32_t  ( *TimeOnAir )( RadioModems_t modem, uint8_t pktLen );
    /*!
     * \brief Sends the buffer of size. Prepares the packet to be sent and sets
     *        the radio in transmission
     *
     * \param [IN]: buffer     Buffer pointer
     * \param [IN]: size       Buffer size
     */
    void    ( *Send )( uint8_t *buffer, uint8_t size );
    /*!
     * \brief Sets the radio in sleep mode
     */
    void    ( *Sleep )( void );
    /*!
     * \brief Sets the radio in standby mode
     */
    void    ( *Standby )( void );
    /*!
     * \brief Sets the radio in reception mode for the given time
     * \param [IN] timeout Reception timeout [ms]
     *                     [0: continuous, others timeout]
     */
    void    ( *Rx )( uint32_t timeout );
    /*!
     * \brief Start a Channel Activity Detection
     */
    void    ( *StartCad )( void );
    /*!
     * \brief Sets the radio in continuous wave transmission mode
     *
     * \param [IN]: freq       Channel RF frequency
     * \param [IN]: power      Sets the output power [dBm]
     * \param [IN]: time       Transmission mode timeout [s]
     */
    void    ( *SetTxContinuousWave )( uint32_t freq, int8_t power, uint16_t time );
    /*!
     * \brief Reads the current RSSI value
     *
     * \retval rssiValue Current RSSI value in [dBm]
     */
    int16_t ( *Rssi )( RadioModems_t modem );
    /*!
     * \brief Writes the radio register at the specified address
     *
     * \param [IN]: addr Register address
     * \param [IN]: data New register value
     */
    void    ( *Write )( uint8_t addr, uint8_t data );
    /*!
     * \brief Reads the radio register at the specified address
     *
     * \param [IN]: addr Register address
     * \retval data Register value
     */
    uint8_t ( *Read )( uint8_t addr );
    /*!
     * \brief Writes multiple radio registers starting at address
     *
     * \param [IN] addr   First Radio register address
     * \param [IN] buffer Buffer containing the new register's values
     * \param [IN] size   Number of registers to be written
     */
    void    ( *WriteBuffer )( uint8_t addr, uint8_t *buffer, uint8_t size );
    /*!
     * \brief Reads multiple radio registers starting at address
     *
     * \param [IN] addr First Radio register address
     * \param [OUT] buffer Buffer where to copy the registers data
     * \param [IN] size Number of registers to be read
     */
    void    ( *ReadBuffer )( uint8_t addr, uint8_t *buffer, uint8_t size );
    /*!
     * \brief Set synchro word in radio
     *
     * \param [IN] data  THe syncword
     */		
		void    ( *SetSyncWord )( uint8_t data );
    
	/*!
     * \brief Sets the maximum payload length.
     *
     * \param [IN] modem      Radio modem to be used [0: FSK, 1: LoRa]
     * \param [IN] max        Maximum payload length in bytes
     */
    void ( *SetMaxPayloadLength )( RadioModems_t modem, uint8_t max );
    
    void ( *OnTimeout)(void);
};
```

+ sx1276的lora事件回调接口
```C
typedef struct
{
    /*!
     * \brief  Tx Done callback prototype.
     */
    void    ( *TxDone )( void );
    /*!
     * \brief  Tx Timeout callback prototype.
     */
    void    ( *TxTimeout )( void );
    /*!
     * \brief Rx Done callback prototype.
     *
     * \param [IN] payload Received buffer pointer
     * \param [IN] size    Received buffer size
     * \param [IN] rssi    RSSI value computed while receiving the frame [dBm]
     * \param [IN] snr     Raw SNR value given by the radio hardware
     *                     FSK : N/A ( set to 0 )
     *                     LoRa: SNR value in dB
     */
    void    ( *RxDone )( uint8_t *payload, uint16_t size, int16_t rssi, int8_t snr );
    /*!
     * \brief  Rx Timeout callback prototype.
     */
    void    ( *RxTimeout )( void );
    /*!
     * \brief Rx Error callback prototype.
     */
    void    ( *RxError )( void );
    /*!
     * \brief  FHSS Change Channel callback prototype.
     *
     * \param [IN] currentChannel   Index number of the current channel
     */
    void ( *FhssChangeChannel )( uint8_t currentChannel );

    /*!
     * \brief CAD Done callback prototype.
     *
     * \param [IN] channelDetected    Channel Activity detected during the CAD
     */
    void ( *CadDone ) ( bool channelActivityDetected );
}RadioEvents_t;
```

+ sx1276的配置参数
    + 顶层封装
```C
typedef struct SX1276_s
{
			uint8_t                   RxTx ; 				//模式选择
			RadioSettings_t       Settings ;			//硬件参数
} SX1276_t;
```
	
 + RadioSettings_t 设置成员
```
typedef struct
{
			RadioState_t                        State;								//模块现状态，RF_IDLE , RF_RX_RUNNING ,RF_TX_RUNNING, RF_CAD
			RadioModems_t                   Modem;							//所处模式， 0 => MODEM_FSK  , 1 => MODEM_LORA
			uint32_t                              Channel;							//通道
			RadioFskSettings_t               Fsk;								//FSK模式的相关设置
			RadioFskPacketHandler_t      FskPacketHandler;			//FSK模式的数据包相关参数
			RadioLoRaSettings_t             LoRa;							//lora模式的相关设置
			RadioLoRaPacketHandler_t    LoRaPacketHandler;		//lora模式的数据包相关参数(rssiValue , Size等)
}RadioSettings_t;
```
	
 + RadioState_t 所处的状态
```
typedef enum
{
			RF_IDLE = 0,
			RF_RX_RUNNING,
			RF_TX_RUNNING,
			RF_CAD,
} RadioState_t;
```
	
 + RadioModems_t 模式
```
typedef enum
{
			MODEM_FSK = 0,
			MODEM_LORA,
} RadioModems_t;
```
	
 + RadioFskSettings_t  FSK的设置
```
typedef struct
{
			int8_t        Power;
			uint32_t     Fdev;
			uint32_t     Bandwidth;
			uint32_t     BandwidthAfc;
			uint32_t     Datarate;
			uint16_t      PreambleLen;
			bool           FixLen;
			uint8_t       PayloadLen;
			bool           CrcOn;
			bool           IqInverted;
			bool           RxContinuous;
			uint32_t     TxTimeout;
}RadioFskSettings_t;
```
	
 + RadioFskPacketHandler_t FSK数据包的相关参数
```
typedef struct
{
			uint8_t    PreambleDetected;
			uint8_t    SyncWordDetected;
			int8_t     RssiValue;
			int32_t    AfcValue;
			uint8_t    RxGain;
			uint16_t   Size;
			uint16_t   NbBytes;
			uint8_t    FifoThresh;
			uint8_t    ChunkSize;
}RadioFskPacketHandler_t;
```
	
 + RadioLoRaSettings_t lora的设置
```
typedef struct
{
			int8_t          Power;
			uint32_t      Bandwidth;
			uint32_t      Datarate;
			bool           LowDatarateOptimize;
			uint8_t       Coderate;
			uint16_t      PreambleLen;
			bool           FixLen;
			uint8_t       PayloadLen;
			bool          CrcOn;
			bool          FreqHopOn;
			uint8_t      HopPeriod;
			bool         IqInverted;
			bool         RxContinuous;
			uint32_t   TxTimeout;
}RadioLoRaSettings_t;
```

 + RadioLoRaPacketHandler_t lora数据包的参数
```
typedef struct
{
			int8_t      SnrValue;
			int16_t     RssiValue;
			uint8_t    Size;
}RadioLoRaPacketHandler_t;
```


## 配置函数

+ 发送配置
```C
 /*
     * \brief Sets the transmission parameters 设置传输的参数
     *
     * \param [IN] modem        Radio modem to be used [0: FSK, 1: LoRa]    // 设置radio模式
     * \param [IN] power          Sets the output power [dBm]						//设置输出功率
     * \param [IN] fdev             Sets the frequency deviation (FSK only)		//设置FSK模式的频率偏移值
     *                                     FSK : [Hz]
     *                                     LoRa: 0
     * \param [IN] bandwidth    Sets the bandwidth (LoRa only)					//设置lora模式下的频宽
     *                          FSK : 0
     *                          LoRa: [  0: 125 kHz, 1: 250 kHz,
     *                                      2: 500 kHz, 3: Reserved ] 
     * \param [IN] datarate     Sets the Datarate										//设置速率SF
     *                          FSK : 600..300000 bits/s
     *                          LoRa: [6: 64, 7: 128, 8: 256, 9: 512,
     *                                10: 1024, 11: 2048, 12: 4096  chips]
     * \param [IN] coderate     Sets the coding rate (LoRa only)				//设置编码率
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [1: 4/5, 2: 4/6, 3: 4/7, 4: 4/8] 
     * \param [IN] preambleLen  Sets the preamble length						//设置前导码的长度
     *                          FSK : Number of bytes 
     *                          LoRa: Length in symbols (the hardware adds 4 more symbols)
     * \param [IN] fixLen       Fixed length packets [0: variable, 1: fixed]			//数据包长度是否固定
     * \param [IN] crcOn        Enables disables the CRC [0: OFF, 1: ON]				//是否校验
     * \param [IN] FreqHopOn    Enables disables the intra-packet frequency hopping		//是否使用调频技术
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [0: OFF, 1: ON]
     * \param [IN] HopPeriod    Number of symbols bewteen each hop			//每一帧间的符号数
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: Number of symbols
     * \param [IN] iqInverted   Inverts IQ signals (LoRa only)						//IQ信号是否反相
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [0: not inverted, 1: inverted]
     * \param [IN] timeout      Transmission timeout [us]							//传输超时时间长度
     */
    void    ( *SetTxConfig )( RadioModems_t modem, int8_t power, uint32_t fdev, 
                              uint32_t bandwidth, uint32_t datarate,
                              uint8_t coderate, uint16_t preambleLen,
                              bool fixLen, bool crcOn, bool FreqHopOn,
                              uint8_t HopPeriod, bool iqInverted);
```

+ 接收配置
```C
/*
     * \brief Sets the reception parameters  设置接受的参数
     *
     * \param [IN] modem        Radio modem to be used [0: FSK, 1: LoRa]    // 设置radio模式
     * \param [IN] bandwidth    Sets the bandwidth									  //设置频宽
     *                          FSK : >= 2600 and <= 250000 Hz
     *                          LoRa: [0: 125 kHz, 1: 250 kHz,
     *                                 2: 500 kHz, 3: Reserved] 
     * \param [IN] datarate     Sets the Datarate											//设置速度
     *                          FSK : 600..300000 bits/s
     *                          LoRa: [6: 64, 7: 128, 8: 256, 9: 512,
     *                                10: 1024, 11: 2048, 12: 4096  chips]
     * \param [IN] coderate     Sets the coding rate (LoRa only)						//设置编码率
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [1: 4/5, 2: 4/6, 3: 4/7, 4: 4/8] 
     * \param [IN] bandwidthAfc Sets the AFC Bandwidth (FSK only) 					//设置AFC带宽
     *                          FSK : >= 2600 and <= 250000 Hz
     *                          LoRa: N/A ( set to 0 ) 
     * \param [IN] preambleLen  Sets the Preamble length								//设置前导码的长度
     *                          FSK : Number of bytes 
     *                          LoRa: Length in symbols (the hardware adds 4 more symbols)
     * \param [IN] symbTimeout  Sets the RxSingle timeout value (LoRa only) 				//设置接受信号的超时时间
     *                          FSK : N/A ( set to 0 ) 
     *                          LoRa: timeout in symbols
     * \param [IN] fixLen       Fixed length packets [0: variable, 1: fixed]					//数据包的长度时候固定
     * \param [IN] payloadLen   Sets payload length when fixed lenght is used			//有效数据包的长度
     * \param [IN] crcOn        Enables/Disables the CRC [0: OFF, 1: ON]				 	//是否数据校验
     * \param [IN] FreqHopOn    Enables disables the intra-packet frequency hopping		//是否使用跳频
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [0: OFF, 1: ON]
     * \param [IN] HopPeriod    Number of symbols bewteen each hop						//每一帧间的符号数
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: Number of symbols
     * \param [IN] iqInverted   Inverts IQ signals (LoRa only)										//IQ信号是否反相
     *                          FSK : N/A ( set to 0 )
     *                          LoRa: [0: not inverted, 1: inverted]
     * \param [IN] rxContinuous Sets the reception in continuous mode					//设置接收模式
     *                          [false: single mode, true: continuous mode]
     */
    void    ( *SetRxConfig )( RadioModems_t modem, uint32_t bandwidth,
                              uint32_t datarate, uint8_t coderate,
                              uint32_t bandwidthAfc, uint16_t preambleLen,
                              uint16_t symbTimeout, bool fixLen,
                              uint8_t payloadLen,
                              bool crcOn, bool FreqHopOn, uint8_t HopPeriod,
                              bool iqInverted, bool rxContinuous );
```







