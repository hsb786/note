@MqProducerStore   标注为生产者

~~~
public @interface MqProducerStore {
	// 发送方式 
	// WAIT_CONFIRM 等待确认(保存消息到数据库中，不发送)  10
	// SAVE_AND_SEND 保存并发送(保存消息到数据库中，并发送)  20
	// DIRECT_SEND 直接发送(不保存消息到数据库中，直接发送)  30
    MqSendTypeEnum sendType() default MqSendTypeEnum.WAIT_CONFIRM;

    MqOrderTypeEnum orderType() default MqOrderTypeEnum.ORDER;
	
    DelayLevelEnum delayLevel() default DelayLevelEnum.ZERO;
}

~~~



@TpcMqMessageService

~~~
// 预存储消息 
saveMessageWaitingConfirm(TpcMqMessageDto) 

// 确认并发送消息
confirmANdSendMessage(messageKey)

// 存储并发送消息
saveAndSendMessage(TpcMqMessageDto)

// 直接发送消息
directSendMessage(TpcMqMessageDto)
~~~

