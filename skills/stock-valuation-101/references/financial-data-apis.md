# A股财务数据快速获取 (Eastmoney API)

> 当 stock-valuation-101 skill 需要获取 A股实时价格和财务数据时使用。

## 实时行情 (Sina API)
```bash
curl -sL "https://hq.sinajs.cn/list=sh600105" \
  -H "Referer: https://finance.sina.com.cn" \
  -H "User-Agent: Mozilla/5.0"
# 返回 var hq_str_sh600105="股票名,今开,昨收,现价,最高,最低,..."
```

## 行情+估值快照 (Eastmoney push API)
```bash
CODE="600105"
curl -sL "https://push2.eastmoney.com/api/qt/stock/get?secid=1.${CODE}&fields=f43,f44,f45,f46,f47,f48,f50,f55,f57,f58,f60,f100,f115,f116,f117,f162,f164,f167,f168,f169,f170,f171" \
  -H "User-Agent: Mozilla/5.0" | python3 -c "
import json,sys
d=json.load(sys.stdin)['data']
print(f'现价:{d[\"f43\"]/100}  PE动:{d.get(\"f115\",0)/100}  总市值:{d.get(\"f116\",0)/1e8:.1f}亿')
print(f'PB:{d.get(\"f167\",0)/100}  换手:{d.get(\"f168\",0)/100}%  涨跌:{d.get(\"f169\",0)/100}%')
"
# 关键字段:
# f43=现价 f115=PE动 f116=总市值 f117=流通市值 f167=PB
# f168=换手率 f169=涨跌幅 f170=涨跌额 f171=振幅
# f57=代码 f58=名称
```

## 季度财务报表 (Eastmoney datacenter API)
```bash
CODE="600105"
curl -sL "https://datacenter.eastmoney.com/securities/api/data/v1/get?reportName=RPT_F10_FINANCE_MAINFINADATA&columns=ALL&filter=(SECURITY_CODE=%22${CODE}%22)&pageNumber=1&pageSize=8&sortColumns=REPORT_DATE&sortTypes=-1" \
  -H "User-Agent: Mozilla/5.0" \
  -H "Referer: https://emweb.securities.eastmoney.com/" | python3 -c "
import json,sys
data=json.load(sys.stdin)['result']['data']
print(f'{\"报告期\":<12} {\"EPS\":>8} {\"扣非EPS\":>8} {\"BPS\":>8} {\"每股FCF\":>8}')
print('-'*55)
for item in data:
    rtype=item['REPORT_TYPE']
    eps=item.get('EPSJB','-')
    eps_kc=item.get('EPSKCJB','-') or '-'
    bps=item.get('BPS','-')
    fcf=item.get('MGWFPLR','-') or '-'
    print(f'{rtype:<12} {eps:>8} {eps_kc:>8} {bps:>8} {fcf:>8}')
"
# 关键字段:
# EPSJB=基本每股收益 EPSKCJB=扣非每股收益 BPS=每股净资产
# MGWFPLR=每股未分配利润(近似FCF) MGZBGJ=每股资本公积金
# REPORT_TYPE=报告类型(一季报/中报/三季报/年报)
```

## 业绩预告 (Eastmoney datacenter API)
```bash
curl -sL "https://datacenter.eastmoney.com/securities/api/data/v1/get?reportName=RPT_PUBLIC_OP_NEWPREDICT&columns=ALL&filter=(SECURITY_CODE=%22${CODE}%22)&pageNumber=1&pageSize=5" \
  -H "User-Agent: Mozilla/5.0" \
  -H "Referer: https://emweb.securities.eastmoney.com/"
# 字段: PREDICT_CONTENT=预告内容 PREDICT_TYPE=预增/预减/扭亏/首亏
#       ADD_AMP_UPPER=增幅上限 PREYEAR_SAME_PERIOD=去年同期净利润
```

## 研报列表 (Eastmoney report API)
```bash
curl -sL "https://reportapi.eastmoney.com/report/list?cb=&industryCode=*&pageSize=5&pageNo=1&code=${CODE}&beginTime=2025-01-01&qType=0&_=$(date +%s)" \
  -H "User-Agent: Mozilla/5.0" \
  -H "Referer: https://data.eastmoney.com/"
# 返回: title=研报标题 orgName=机构 rating=评级 reportDate=日期
```

## 季报EPS拆解技巧

季报披露的 EPS 是**累计值**而非单季值。必须逐季拆解：

```python
# 示例: 永鼎股份2025年
# 一季报   0.1982  → Q1 = 0.1982
# 中报     0.218   → Q2 = 0.218 - 0.1982 = 0.0198  (暴降90%!)
# 三季报   0.225   → Q3 = 0.225 - 0.218  = 0.007   (几乎不赚钱)
# 年报     0.1598  → Q4 = 0.1598 - 0.225 = -0.0652 (亏损!)
```

这种「Q1脉冲→Q2-Q4恶化」的模式是典型的价值陷阱信号——全年利润几乎全来自Q1，后续逐季走弱。发现此类模式时应重点核查 Q1 利润是一次性的（投资收益/资产处置）还是持续性的（主业改善）。
