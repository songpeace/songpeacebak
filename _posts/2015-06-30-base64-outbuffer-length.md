---
layout: post
category: "android"
title: "C语言Base64输出buffer大小计算"
tags: ["C语言,Base64,输出,buffer,大小"]
---
####一、Base64编码过程
先按3字节分块（3×8=24)，再切成(4×6，高位补0)大小4字节，即3×8=4×6。简言之，3个字节的输入会变成4个字节的输出。  
因此，输入字符串长度如果不是3的整数倍，则需要在末尾补1或2个0。  

####二、C语言实现计算输出buffer(char *)大小计算方法
	/**
	* 根据原始输入字符串计算编码后字符串的长度
	*/
	int getOutputLen(unsigned  char * input) {
		int orgLen = strlen(input);
		int addLen = (orgLen % 3 == 0) ? 0 : (3 - orgLen % 3);
		int result = orgLen + addLen;
		return  result*8 / 6;
	}
	
	/**
	* 根据编码后的字符串计算编码前字符串的长度（补位后的）
	*/
	int getInputLen(unsigned  char * input) {
	    unsigned char *p =input;
	    int count=0;
	    while(*p!='\0' && *p!='\n') {//过滤\n
	        count++;
	        p++;
	    }
	    return count*6/8;
	}

####三、编码中有\n字符的情况
JAVA base64的某些实现版本，编码后的字符串中可能带有"\n"字符（方便输出时查看），getOutputLen就不再准确了，需要过滤掉"\n"字符。

	System.out.println("getOutputLen:"+getOutputLen(input));//根据原始输入字符串计算编码后字符串的长度为1144
	String result = Base64.encodeToString(input.getBytes("UTF-8"), 0);//对字符串按Base64编码
	System.out.println("result Len:"+result.length());//实际的result长度发现为1160
	result=result.replaceAll("\n", "");//因为它附加了\n字符，需要滤\n字符
	System.out.println("real result Len:"+result.length());//输出1144，这就对了！

（完~）
