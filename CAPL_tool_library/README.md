# CAPL tool library V2.0

- Encapsulate diagnostic related functions based on osektp.dll
- Enhance common functions such as printing, output and other functions

## Function
- [x] Diagnostic send and receive

- [ ] assertion (partial completion)

- [ ] File parsing (partially completed)

- [ ] Package fault injection

- [ ] Beautify test report

- [x] Printing enhancements

- [ ] Commonly used test functions

- [ ] ...



## Latest updates

*Updated to version 2.0 on September 8, 2023
   - [x] Optimize the diagnostic data storage structure
   - [x] Fixed diagnostic response timeout when sending multiple frames
   - [x] Add logging related functions to beautify printing
   - [x] Optimize outputPro related logic

- August 20, 2023 Updated node loss related DTC detection tool (demo)
- August 17, 2023 Updated diagnostic non-response related functions

  

------

  

# Quick start

> This tool is strongly related to osektp.dll, please be sure to configure osektp.dll in the node
> If you want to use 27 unlocking related functions, please ensure that the CANoe diagnostic panel can be unlocked normally.



### Create connection

```c
variables
{
  long handle;
  char ecuName[10] = "RLDHC";
  byte lv1=0x01;
  byte lvFbl = 0x09;
}
MainTest(){
  handle = CreateCANFDConnection(0x746,0x74E);
    // or CreateCANConnection(0x746,0x74E);
}
```



### Send diagnostics and assertions

```c
//Send 10 01
sendDiag1001(); 

// Determine whether the response is a positive response
AssertDiagTrue(); 

// Determine whether the 4th byte of the response data is 0
AssertDiagCompareBytes(4,0x0);

// Send key 0000
sendDiag27_key(handle,lv1,0x00000000);

// Determine whether it is a negative response, and the response code is 0x24
AssertDiagFalseWithNRC(0x24);

// 解锁密钥
sendDiag27_UnlockSecurityLevel(handle,lv1,ecuName);

// 22服务读取DID
int len;
byte buffer[100];
sendDiag22xxxx(diagHandle,0x0200);
len = getDiagResp(buffer); // 获取诊断响应数据
```

```c
testcase DFRTC005(){
  word segmentSize,len;
  byte buffer[100];
  
  testCaseTitle("DFRTC005","刷新块大小参数测试");
  testCaseDescription("校验 ECU 刷新块大小参数的准确性");
 
  Pre_Programming();
  if (TestGetVerdictLastTestCase() == 1)
    return;
  sendDiag1002(diagHandle);
  AssertDiagTrue();
  sendDiag27_UnlockSecurityLevel(diagHandle,lvFbl,ecuName);
  sendDiag(diagHandle,q2EF184);
  AssertDiagTrue();
  if (TestGetVerdictLastTestCase() == 1)  //如果测试用例已经失败，则停止继续执行
    return;
  memcpy_off(q340044,3,gacDriverBin.AddressRegion[0],0,8);
  sendDiag(diagHandle,q340044);
  AssertDiagTrue();
  len=getDiagResp(buffer); // 获取34应答中的传输数据大小
  segmentSize = buffer[len-2];
  segmentSize <<= 8;
  segmentSize +=buffer[len- 1];
  // 测试传输数据超过预期
  sendDiag36(diagHandle,gacDriverBin.fileBlock[0],gacDriverBin.fileBlockLen[0],0,segmentSize+2);
  AssertDiagFalseWithNRC(0x13);
  //测试传输数据是预期的一半
  fbl(gacDriverBin,gacAppBin,(segmentSize/2+1));
  //测试传输数据大小为128
  fbl(gacDriverBin,gacAppBin,130);
}
```



##?

If you have any questions, you can submit a PR and it will definitely be changed, 
but it’s hard to say when.

Since I have only been working for a few months and have a serious lack of experience, 
there are definitely bugs and unreasonable things in the code. I hope you can give me some pointers.

It would be better if there are friends who can improve it with me.