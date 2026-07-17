# Financial Data Sources for Valuation

## API Endpoints (use with Python urllib)

### Real-time Market Data
```
https://push2.eastmoney.com/api/qt/stock/get?secid={market}.{code}&fields=f43,f57,f58,f115,f116,f117,f167
```
- market: 1=Shanghai, 0=Shenzhen
- f43=price, f115=PE, f116=market cap, f117=float cap, f167=PB

### Quarterly Financial Data
```
https://datacenter.eastmoney.com/securities/api/data/v1/get?reportName=RPT_F10_FINANCE_MAINFINADATA&columns=REPORT_DATE,REPORT_TYPE,EPSJB,EPSKCJB,BPS,MGWFPLR,PARENT_NETPROFIT,TOTAL_OPERATE_INCOME&filter=(SECURITY_CODE="股票代码")&pageNumber=1&pageSize=8&sortColumns=REPORT_DATE&sortTypes=-1
```
- EPSJB=basic EPS, EPSKCJB=deducted EPS, BPS=book value per share
- MGWFPLR=undistributed profit per share (proxy for cumulative retained earnings)
- PARENT_NETPROFIT=net profit attributable to parent
- ⚠️ A-share quarterly EPS is CUMULATIVE — must decompose single-quarter EPS manually
- ⚠️ API rate limits: space out calls by 2+ seconds

### Real-time Sina (no rate limit, simpler)
```
https://hq.sinajs.cn/list={prefix}{code}
```
- Prefix: sh=Shanghai, sz=Shenzhen
- Returns GBK-encoded CSV; use `iconv -f GBK -t UTF-8` to decode

## Search for Latest Earnings Forecasts

### Priority 1: Google News RSS (most reliable for Chinese financial news)
```
https://news.google.com/rss/search?q={股票名称}+{代码}+业绩+预告&hl=zh-CN&gl=CN&ceid=CN:zh-Hans
```
- Returns structured XML with titles; parse with regex `<title>([^<]*)</title>`
- Best for finding: earnings pre-announcements, analyst reports, 财联社 coverage

### Priority 2: DuckDuckGo (when Google is blocked)
```
https://duckduckgo.com/html/?q={股票名称}+{代码}+中报+预告+2026+业绩
```
- Strip HTML tags with `sed 's/<[^>]*>//g'`
- Less reliable than Google News RSS but works more consistently from server environments

### Key Search Patterns
- `{股票名}+中报预告+2026+业绩` — mid-year report preview
- `{股票名}+业绩预增+预减` — profit warning
- `site:cls.cn+{股票名}+业绩` — 财联社 specific
- `{股票名}+{代码}+研报+目标价` — analyst reports and price targets

## Bilibili Video Download (for render-pdf skill)

### Metadata
```
https://api.bilibili.com/x/web-interface/view?bvid={BV}
```

### Audio Download
```bash
yt-dlp -f "worstaudio" --extract-audio --audio-format wav \
  --user-agent "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  --add-header "Referer:https://www.bilibili.com" \
  --add-header "Origin:https://www.bilibili.com" \
  --extractor-args "bilibili:player_args=hardware_acceleration=1" \
  -o "audio.%(ext)s" "<URL>"
```

### Whisper Transcription (whisper.cpp)
```bash
cd /tmp/whisper.cpp
./build/bin/whisper-cli \
  -m models/ggml-tiny.bin \
  -f audio.wav \
  -l zh \
  -osrt \
  -of output_prefix
```
