---
title: l2cap
tags: 
---

### GAPM\_LE\_CREDIT\_PSM\_CREATE\_CMD

**Procedure:**

s_xxx means server with PSM 
c_xxx means client 


```sequence

participant s_app
participant s_stack
s_app->s_stack:GAPM_LE_CREDIT_PSM_CREATE_CMD
s_stack->s_app:GAPM_CMP_EVT

```


**API:**

`void app_gapm_le_credit_psm_create_cmd(uint16_t le_psm, uint16_t mps_len,uint16_t sec_lvl, uint16_t intial_credit)`

|Type|Parameters|Description|
|-|-|-|
|uint16_t |le_psm|0x0080-0x00FF
|uint16\_t | mps\_len|The maximum size of payload data。当前实现中此值不能超过MTU
|uint16_t | sec\_lvl||
|uint16_t |initial\_credit|指明对等设备可以向自己发送的数据包数0-65535<br>The initial credit value indicates the number of LE-frames that the peer device can send to the L2CAP layer entity sending the LE Credit Based Connection Request. The initial credit value shall be in the range of 0 tob65535.

**Response**:
GAPM_CMP_EVT：operation，status

* 成功：status = GAP\_ERR\_NO\_ERROR
* 如果le\_psm已经注册过，status=GAP\_ERR\_COMMAND\_DISALLOWED 
* 如果le\_psm值超过范围 status = GAP\_ERR\_INVALID_PARAM
* 如果已经达到内部可以创建的最大个数 status=GAP\_ERR\_NOT\_SUPPORTED,最大值为GAP\_LEPSM\_CNX\_MAX

**le_psm:** 0x0080-0x00FF

---


### GAPM_LE_CREDIT_PSM_DESTROY_CMD

**Procedure:**

```sequence

participant s_app
participant s_stack

s_app->s_stack: GAPM_LE_CREDIT_PSM_DESTROY_CMD
s_stack->s_app:GAPM_CMP_EVT


```

**API:**
|Type|Parameters|Description|
|-|-|-|
|uint16_t|le\_psm|LE Protocol/Service Multiplexer|

**Response**:
GAPM_CMP_EVT：operation，status

* 成功：status = GAP\_ERR\_NO\_ERROR
* 如果未创建过这个le\_psm,status = GAP\_ERR\_NOT\_FOUND
* 如果这个le\_psm上还有未断开的channel，不进行销毁动作，status = GAP\_ERR\_COMMAND\_DISALLOWED

---

### GAPC\_LE\_CREDIT\_CON\_CONNECT\_CMD
**Procedure:**
如果协议栈检查参数有误:
```sequence

participant s_app
participant s_stack
participant c_stack
participant c_app
s_app->s_stack:GAPM_LE_CREDIT_PSM_CREATE_CMD
s_stack->s_app:GAPM_CMP_EVT

```


如果对方设备在规定是间内没有回应:
```sequence

participant s_app
participant s_stack
participant c_stack
participant c_app
s_app->s_stack:GAPM_LE_CREDIT_PSM_CREATE_CMD

s_stack-->c_stack:LE CREDIT BASED CONNECTION REQUEST

s_stack->s_app:GAPC_LECB_CONN_TO_IND

```
如果对方发现参数不合适：
```sequence

participant s_app
participant s_stack
participant c_stack
participant c_app
s_app->s_stack:GAPM_LE_CREDIT_PSM_CREATE_CMD

s_stack-->c_stack:LE CREDIT BASED CONNECTION REQUEST

s_stack->s_app: GAPC_LE_CREDIT_CON_CONNECT_IND

```
如果对方检查参数正确并做出回应：
```sequence

participant s_app
participant s_stack
participant c_stack
participant c_app

note left of s_app: app_gapc_le_credit_con_connect_cmd
s_app->s_stack:GAPM_LE_CREDIT_PSM_CREATE_CMD

s_stack-->c_stack:LE CREDIT BASED CONNECTION REQUEST

c_stack->c_app: GAPC_LE_CREDIT_CON_CONNECT_REQ_IND
note right of c_app: app_gapc_le_credit_con_connect_cfm
c_app->c_stack: GAPC_LE_CREDIT_CON_CONNECT_CFM

c_stack-->s_stack: LE CREDIT BASED CONNECTION RESPONSE

s_stack->s_app: GAPC_LE_CREDIT_CON_CONNECT_IND

```

**API:**
|Type|Parameters|Description|
|-|-|-|
|uint8_t|conidx|LE Protocol/Service Multiplexer|
|uint16_t|le\_psm|LE Protocol/Service Multiplexer|
|uint16_t|credit|指明对等设备可以向自己发送的数据包数0-65535|
|uint16_t|mps_len|The maximum size of payload data。当前实现中此值不能超过MTU|

**Response:**
发起方GAPC_CMP_EVT: operation，status

* 如果le_psm范围不在0x0080-0x00FF，status=GAP_ERR_INVALID_PARAM
* 如果CID已经用尽，status = GAP_ERR_COMMAND_DISALLOWED

发起方GAPC_LECB_CONN_TO_IND

* 如果GAP_TMR_LECB_CONN_TIMEOUT时长用尽，但仍没收到对方响应，则回复此消息

发起方收到GAPC_LE_CREDIT_CON_CONNECT_IND

* 如果此命令得到对方响应则自己会收到此消息 

响应方收到GAPC_LE_CREDIT_CON_CONNECT_REQ_IND

* 响应方收到连接请求消息，协议栈检查通过后向上层发送此消息


---
### GAPC_LE_CREDIT_CON_CONNECT_REQ_IND

**API:**
```
int app_gapc_le_credit_con_connect_req_ind_handler(ke_msg_id_t const msgid,
                    struct gapc_le_credit_con_connect_req_ind const *param,
                    ke_task_id_t const dest_id, ke_task_id_t const src_id)
```

这条消息是发送给上层，表名有设备向自己发送建立channel请求，并附带参数，参数如下：

```
/// Parameters of the @ref GAPC_LE_CREDIT_CON_CONNECT_REQ_IND message
struct gapc_le_credit_con_connect_req_ind
{
    /// LE Protocol/Service Multiplexer
    uint16_t le_psm;
    /// Destination Credit for the LE Credit Based Connection
    uint16_t dest_credit;
    /// Maximum SDU size
    uint16_t max_sdu;
    /// Source channel ID
    uint16_t src_cid;
    /// Destination CID
    uint16_t dest_cid;
};
```
其中 `src_cid` 是协议栈分配给自己的cid，`dst_cid` 是指请求方的cid。
此消息需要app处理，是否通知协议栈允许此channel建立的决定权在app。

---
### GAPC_LE_CREDIT_CON_CONNECT_IND

发送建立channel请求的设备在收到对方回应后向上层发送此消息
```
/// Parameters of the @ref GAPC_LE_CREDIT_CON_CONNECT_IND message
struct gapc_le_credit_con_connect_ind
{
    /// Destination Credit for the LE Credit Based Connection
    uint16_t dest_credit;
    /// Maximum SDU size
    uint16_t max_sdu;
    /// Destination CID
    uint16_t dest_cid;
    /// src CID
    uint16_t src_cid;

};
```
其中 `src_cid` 是协议栈分配给自己的cid，`dst_cid` 是指请求方的cid。

---
### GAPC_LE_CREDIT_CON_CONNECT_CFM

**API:**
`void app_gapc_le_credit_con_connect_cfm(ke_task_id_t const dest_id, uint16_t src_cid, uint16_t credit, uint8_t status)`

拥有PSM服务的应用收到**GAPC_LE_CREDIT_CON_CONNECT_REQ_IND** 可回复此消息。

---
### GAPC_LE_CREDIT_DISCONNECT_CMD

**Procedure:**
如果协议栈检查参数有误：
```sequence

participant s_app
participant s_stack
participant c_stack
participant c_app
s_app->s_stack: GAPC_LE_CREDIT_DISCONNECT_CMD
s_stack->s_app: GAPM_CMP_EVT
```

如果对方在规定时间内无响应：
```sequence

participant s_app
participant s_stack
participant c_stack
participant c_app
s_app->s_stack: GAPC_LE_CREDIT_DISCONNECT_CMD
s_stack-->c_stack: DISCONNECTION REQUEST

s_stack->s_app: GAPC_LECB_DISCONN_TO_IND

```

如果对方响应：
```sequence

participant s_app
participant s_stack
participant c_stack
participant c_app
s_app->s_stack: GAPC_LE_CREDIT_DISCONNECT_CMD
s_stack-->c_stack: DISCONNECTION REQUEST

c_stack->c_app: GAPC_LE_CREDIT_DISCONNECT_IND
c_stack-->s_stack: L2C_DISCONNECTION_RESP
s_stack->s_app: GAPC_LE_CREDIT_DISCONNECT_IND
```
**API:**
`void app_gapc_le_credit_disconnect_cmd(uint8_t conidx, uint16_t cid)`
|Type|Parameters|Description|
|-|-|-|
|uint8\_t | conidx|
|uint16\_t | cid|自己的cid

**Response:**
发起方GAPC_CMP_EVT: operation，status

* 如果协议栈检查参数有误
* 如果cid没有连接，status=GAP_ERR_NOT_FOUND

发起方GAPC_LECB_DISCONN_TO_IND
双方GAPC_LE_CREDIT_DISCONNECT_IND

* 断开channel成功


---
### GAPC_LE_CREDIT_DISCONNECT_IND

```
struct gapc_le_credit_disconnect_ind
{
    /// cid
    uint16_t cid;
    /// Reason
    uint16_t reason;
}
```
|Type|Parameters|Description|
|-|-|-|
|uint16\_t | cid | 自己的CID
|uint16\_t | reason| 断开的原因


---

### GAPC_LE_CREDIT_CON_ADD_CMD

**Procedure:**

```sequence

participant s_app
participant s_stack
participant c_stack
participant c_app
s_app->s_stack: GAPC_LE_CREDIT_DISCONNECT_CMD
s_stack->s_app: GAPM_CMP_EVT
s_stack-->c_stack: LE FLOW CONTROL CREDIT
c_stack->c_app: GAPC_LE_CREDIT_CON_CONNECT_IND
```
**API:**
`void app_gapc_le_credit_con_add_cmd(uint8_t conidx, uint16_t cid, uint16_t credit)`
|Type|Parameters|Description|
|-|-|-|
|uint8\_t | conidx|
|uint16\_t | cid|自己的cid
|uint16\_t | credit| 增加的credit数目


---

### GAPC_LE_CREDIT_CON_ADD_IND


```
struct gapc_le_credit_con_add_ind
{
    /// src cid
    uint16_t src_cid;
    /// Source Credit for the LE Credit Based Connection
    uint16_t src_credit;
    /// Destination Credit for the LE Credit Based Connection
    uint16_t dest_credit;
};
```
|Type|Parameters|Description|
|-|-|-|
|uint16\_t | src_cid| 自己的cid
|uint16\_t | src_credit | 自己现有的credit
|uint16\_t | dest_credit | 对方的credit


---

### L2CC_PDU_SEND_REQ

**Procedure:**
```sequence
note left of s_app: app_l2cap_data_send_req
s_app->s_stack:L2CC_PDU_SEND_REQ
s_stack->s_app: GAPC_CMP_EVT

c_stack->c_app: L2CC_LECNX_DATA_RECV_IND
note right of c_app: app_l2cap_data_rcv_ind_handler
```

**API:**
`void app_l2cap_data_send_req(uint8_t conidx, uint16_t dest_cid, uint16_t data_size, uint8_t *pdata)`

|Type|Parameters|Description|
|-|-|-|
|uint8\_t | conidx| 
|uint16\_t | dest_cid| 要发送到的cid
|uint16\_t | data_size| 要发送的数据长度
|uint8\_t * | pdata| 要发送的数据起始地址


--- 
### L2CC_LECNX_DATA_RECV_IND
收到此消息表示底层收到数据，有设备向某CID发送数据

```
struct l2cc_lecnx_data_recv_ind
{
    /// Source channel ID
    uint16_t src_cid;
    /// Source remaining credit
    uint16_t src_credit;
    /// Data length
    uint16_t len;
    /// data
    uint8_t data[__ARRAY_EMPTY];
};
```
|Type|Parameters|Description|
|-|-|-|
|uint16\_t | src_cid| 收到数据的CID
|uint16\_t | src_credit| 本CID可用的credit
|uint16\_t | len| 收到的数据长度
|uint8\_t | data| 收到的数据起始地址


---


### L2CC_PDU_SEND_RSP


```
struct l2cc_data_send_rsp
{
    /// Status of request.
    uint8_t status;
    /// Destination channel ID
    uint16_t dest_cid;
    /// Destination credit
    uint16_t dest_credit;
}
```
|Type|Parameters|Description|
|-|-|-|
|uint8\_t  | status| 协议栈检查各种参数结果
|uint16\_t | dest_cid | 要发到某个CID
|uint16\_t | dest_credit | dest\_cid剩余的credit 



---





















































