
> [!NOTE] 自回归的特性只体现在推理
> 训练时输出`(b,l,V)`，展平成`(b*l,V)`，标签也展平`(b*l)`，这样就是分类问题的损失函数
> 
> 推理时单样本输出`(1,l,d)`，只取最后一组概率的最大，追加到输入，这才体现出了自回归性质

### 注意力

![多头自注意力示意图](https://img.baidu.re/i/2026/04/qnr2rn.jpg)

- 输入：dec_output
- 输出：attn_output
- 权重：W_Q/K/V/O

```py
class Attention(nn.Module):
  def __inti__(self):
  supe(Attention,self).__inti__()
  self.W_Q=nn.Linear(d_model,d_model)
  self.W_K=nn.Linear(d_model,d_model)
  self.W_V=nn.Linear(d_model,d_model)
  self.W_O=nn.Linear(d_model,d_model)
  
  def forward (self,dec_output):
 
# 施加权重并拆分多头
Q=self.W_Q(dec_output).view(batch_size,-1,n_head,d_k).transpose(1,2)
    K=self.W_K(dec_output).view(batch_size,-1,n_head,d_k).transpose(1,2)
    V=self.W_V(dec_output).view(batch_size,-1,n_head,d_k).transpose(1,2)
    
    #此时Q，K，V均为4D张量（batch_size,n_head,len_seq,d_k）

score=Q@K.transpose(-2,-1))

# 掩码
mask=np.triu(np.ones([len_seq,len_seq]),k=1)

mask=torch.from_numpy(mask).byte()
    
score.masked_fill_(mask,-1e9)/torch.sqrt(d_k)
       context=nn.Softmax(dim=-1)(score)@V
    
# 多头重组
context=context.transpose(1,2).contiguous().view(batch_size,-1,d_model)

#施加权重并输出
attn_output=self.W_O(attention)
    
    return attn_output
```


### 前馈

- 输入：input
- 输出：output
- 权重：卷积

```py
class FFN(nn.Module):
  def __init__(self):
    super(FFN,self).__init__()
    self.conv1=nn.Conv1d(in_channels=d_model,out_channels=2048,kernel_size=1)
    self.conv2=nn.Conv1d(in_channels=2048,out_channels=d_model,,kernel_size=1)
    
  def forward(self,input):
    output=nn.ReLU()(self.conv1(input.transpose(1,2)))
    ffn_output=self.conv2(output).transpose(1,2)
    
    return ffn_output
```


### 解码器

单层：

- 输入：dec_output
- 输出：dec_output
- 权重：归一化
- 过程：输入→注意力→加和归一→前馈→加和归一→输出

```py
class DecoderLayer(nn.Module):
  def __init__(self):
    super(DecoderLayer,self).__init__()
    self.attn=Attetion()
    self.ffn=FFN()
    self.norm1=nn.LayerNorm(d_model)
    self.norm2=nn.LayerNorm(d_model)
    
    def forward(self,dec_output):
      attn_output=attn(dec_output)
      norm1_output=self.norm1(attn_output+dec_output)
      ffn_output=self.ffn(norm1_output)  
      dec_output=self.norm2(ffn_output+norm1_output)
      return dec_output
```

串联多层：

- 输入：dec_output
- 输出：dec_output
- 权重：无，循环多层

```py
class Decoder(nn.Module):
  def __int__(self):
    super(Decoder,self).__init__()
    self.layers=nn.ModuleList([DecoderLayer() for _ in range(6)]) #6个解码器层
    
    def forward(self,dec_output):
     for layer in layers:
       dec_output=layer(dec_output)
    
    return dec_output
```



### 嵌入

- 输入：一批词元索引列表
- 输出：词和位置嵌入的3D张量
- 权重：嵌入查找表


```py
class Emdedding(nn.Module):
  def __init__():
    super(Embbeding,self).__init__()
    self.token_emb=nn.Embedding(vocab,d_model)
  
  def forward(self,input):
    pos_emb=np.zeros(len_seq,d_model)
    angle=[pos/(1000^(2i/d_model)) for pos in len_seq for i in d_model]
    pos_emb[:,0::2]=np.sin(angle[:,0::2])
    pos_emb[:,1::2]=np.cos(angle[:,1::2])
    return self.token_emb(inputs)+pos_emb
```


### 分类

- 输入：dec_output
- 输出：一批多组类别概率
- 权重：维度转类别的矩阵

```py
class Classification(nn.Module):
  def __init__():
    super(Classification,self).__init__()
    self.projection=nn.Linear(d_model,vocab)
    
    def forwad(self,dec_output):
     return nn.Softmax(dim=-1)(self.projection(dec_output))
```

### 模型

- 输入：一批索引列表
- 输出：一批多组类别概率

```py
class Model(nn.Module):
  def __init__(self):
    super(model,self).__init__()
    self.embbeding=Embbeding()
 self.decoder=Decoder()   self.classification=Classifiction()
 
 def forward(self,input):
    dec_output=embbeding(input)
    dec_output=decoder(dec_output)
    
  output=classification(dec_output)  
```


### 训练

```py
input torch.optim as optim
device="cuda" if torch.cuda.is_available() else "cpu"
model=Model(vocab_size,max_seq_len).to(device)
criterion=nn.CrossEntropyLoss()#损失函数
optimizer=optim.Adam(model.parameters(),lr=0.0001)#优化器
epochs=500#训练轮次
for epoch in range(epochs):
  optimizer.zero_grad()
  inputs,targets=corpus.make_batch(batch_size)
  inputs,targets=inputs.to(device),targets.to(device)
  outputs=model (inputs)
  loss=criterion(outputs.view(-1,vocab),targets.view(-1))
  if(epoch+1)%100==0:#每100轮打印一次损失
    print(f"Epoch:{epoch+1:04d} cost={loss:6f}")
  loss.backward()
  optimizer.step()
```

### 推理

```py
def generate_text(model,input_str,max_len=50)
   model.eval()#测试模式
   #词元转索引
   input_tokens=[corpus.vocab[token] for  token in input_str]
   #复制到新列表      
   output_tokens=input_tokens.copy()
   with torch.no_grad():
     for _ in range (max_len): 
               inputs=torch.LongTensor(output_tokens).unsqueeze(0).to(device)
       #输出形状为[1,len(output_tokens),vocab_size]
       outputs=model(inputs)
       #注意这里，在最后一个维度上获取最大值，并返回索引
       _,next_token=torch.max(outputs [:,-1,:],dim=-1)
      next_token=next_token.item()
      if next_token==corpus.vocab("<eos>"):
        break
      output_tokens.append(next_token)
  output_str="".join([corpus.idx2word[token] for token in output_tokens])
 return output_str
 
 
iniput_str=["Python"]
genetated_text=generated_text(model,input_str)
print ("生成的文本",generated_text)
```


