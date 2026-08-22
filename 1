# -*- coding:utf-8 -*-
import json
import time
import os
import logging
import datetime
import threading
import akshare as ak
import requests
from flask import Flask, render_template_string

app = Flask(__name__)

# ==========配置============
TARGET_CODE = "513390"
TARGET_NAME = "纳指ETF博时"
THRESHOLD_DISCOUNT = 0
THRESHOLD_GOOD = 3
THRESHOLD_OBSERVE = 5
SUMMARY_HOUR =17
CHECK_INTERVAL =60
CONFIG_FILE="config.json"
LOG_PATH="./logs"
os.makedirs(LOG_PATH, exist_ok=True)
# ==========================

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s",
    handlers=[
        logging.FileHandler(os.path.join(LOG_PATH,"app.log"), encoding="utf-8"),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger(__name__)

latest_data = {"ok":False,"msg":"等待首次拉取"}
log_buffer = []

def load_config():
    if not os.path.exists(CONFIG_FILE):
        default_cfg = {"feishu_webhook":"","enable_feishu":True}
        with open(CONFIG_FILE,"w",encoding="utf8") as f:
            json.dump(default_cfg,f,indent=2,ensure_ascii=False)
    with open(CONFIG_FILE,"r",encoding="utf8") as f:
        return json.load(f)

cfg = load_config()

def add_log(txt):
    t = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    item = f"[{t}] {txt}"
    log_buffer.append(item)
    if len(log_buffer)>150:
        log_buffer.pop(0)
    logger.info(txt)

def send_feishu(text):
    if not cfg.get("enable_feishu") or not cfg.get("feishu_webhook"):
        return
    try:
        payload = {"msg_type":"text","content":{"text":text}}
        requests.post(cfg["feishu_webhook"],json=payload,timeout=8)
        add_log("飞书消息已发送")
    except Exception as e:
        add_log(f"飞书推送失败:{str(e)}")

def get_trade_calendar():
    try:
        df = ak.tool_trade_date_hist_sina()
        return set(df["trade_date"].tolist())
    except Exception as e:
        add_log(f"交易日历获取异常 {e}")
        return set()

def is_today_trade_day(trade_set):
    today = datetime.date.today()
    return today in trade_set

def is_in_trade_time():
    now = datetime.datetime.now()
    hh, mm = now.hour, now.minute
    morning = (hh ==9 and mm >=30) or hh ==10 or (hh ==11 and mm <=30)
    afternoon = 13 <= hh < 15
    return morning or afternoon

def fetch_513390():
    res = {"ok":False}
    try:
        df = ak.fund_etf_spot_em()
        row = df[df["代码"] == TARGET_CODE]
        if row.empty:
            res["err"] = "无行情数据"
            return res
        r = row.iloc[0]
        price = float(r["最新价"])
        iopv = float(r["IOPV"])
        amount = float(r["成交额"])
        dt_str = str(r["行情时间"])
        if iopv ==0:
            res["err"]="IOPV等于0"
            return res
        premium = (price - iopv)/iopv*100
        res.update({
            "ok":True,
            "price":round(price,3),
            "iopv":round(iopv,3),
            "premium":round(premium,2),
            "amount":amount,
            "data_time":dt_str
        })
        return res
    except Exception as e:
        res["err"]=str(e)
        return res

def get_tag(premium):
    if premium < THRESHOLD_DISCOUNT:
        return "折价"
    elif premium < THRESHOLD_GOOD:
        return "较理想关注区间"
    elif premium < THRESHOLD_OBSERVE:
        return "可以观察"
    else:
        return "未触发"

def monitor_loop():
    global latest_data
    add_log("后台监控线程已启动")
    send_feishu("✅云监控已启动，监控513390纳指ETF博时")
    trade_dates = get_trade_calendar()
    total_check =0
    last_summary_date=None
    while True:
        try:
            today = datetime.date.today()
            if not is_today_trade_day(trade_dates):
                add_log("非交易日，休眠60秒")
                time.sleep(60)
                continue
            now_time = datetime.datetime.now()
            if now_time.hour == SUMMARY_HOUR and last_summary_date != today:
                snap = fetch_513390()
                lines = [f"【513390每日收盘总结 {today}】",f"累计轮询次数:{total_check}"]
                if snap["ok"]:
                    tag=get_tag(snap["premium"])
                    lines.append(f"现价:{snap['price']} IOPV:{snap['iopv']} 溢价:{snap['premium']}% 状态:{tag}")
                else:
                    lines.append(f"行情异常:{snap.get('err')}")
                report = "\n".join(lines)
                send_feishu(report)
                last_summary_date=today

            if not is_in_trade_time():
                time.sleep(30)
                continue

            total_check +=1
            snap_data = fetch_513390()
            latest_data = snap_data
            if not snap_data["ok"]:
                add_log(f"行情拉取失败:{snap_data.get('err')}")
                time.sleep(CHECK_INTERVAL)
                continue
            premium = snap_data["premium"]
            tag_text = get_tag(premium)
            add_log(f"轮询#{total_check} 溢价:{premium}% {tag_text}")
            if premium < THRESHOLD_OBSERVE:
                alert = f"⚠️513390溢价告警\n{TARGET_CODE} {TARGET_NAME}\n现价:{snap_data['price']}\nIOPV:{snap_data['iopv']}\n溢价率:{premium}%\n状态:{tag_text}\n行情时间:{snap_data['data_time']}"
                send_feishu(alert)
            time.sleep(CHECK_INTERVAL)
        except Exception as e:
            add_log(f"监控线程异常:{str(e)}")
            time.sleep(10)

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>513390纳指ETF博时溢价监控</title>
<style>
*{box-sizing:border-box;margin:0;padding:0;font-family:system-ui,-apple-system,sans-serif;}
body{background:#121212;color:#eee;padding:14px;max-width:640px;margin:0 auto;}
.card{background:#1e1e1e;border-radius:12px;padding:16px;margin-bottom:12px;}
.big-val{font-size:32px;font-weight:bold;margin:10px 0;}
.tag{padding:6px 12px;border-radius:8px;display:inline-block;font-weight:bold;}
.tag-discount{background:#00695c;}
.tag-good{background:#0277bd;}
.tag-observe{background:#ef6c00;}
.tag-none{background:#616161;}
.row{display:flex;justify-content:space-between;padding:7px 0;border-bottom:1px solid #333;}
.lbl{color:#aaa;}
.log{background:#0a0a0a;padding:10px;border-radius:8px;max-height:240px;overflow:auto;font-size:12px;white-space:pre-wrap;color:#ccc;}
</style>
</head>
<body>
<div class="card">
<h2>513390｜纳指ETF博时</h2>
{% if data.ok %}
<div class="big-val">{{data.premium}} %</div>
{% set tagname = get_tag_name(data.premium) %}
<div class="tag {{tagname[1]}}">{{tagname[0]}}</div>
{% else %}
<div class="big-val">--</div>
<div class="tag tag-none">{{data.msg}}</div>
{% endif %}
</div>
<div class="card">
<div class="row"><span class="lbl">现价</span><span>{{data.price if data.ok else '-'}}</span></div>
<div class="row"><span class="lbl">IOPV</span><span>{{data.iopv if data.ok else '-'}}</span></div>
<div class="row"><span class="lbl">成交额</span><span>{{(data.amount/10000)|round(2)}}万</span></div>
<div class="row"><span class="lbl">行情时间</span><span>{{data.data_time if data.ok else '-'}}</span></div>
</div>
<div class="card">
<h3>运行日志</h3>
<div class="log">
{% for l in logs %}
{{l}}
{% endfor %}
</div>
</div>
<script>
setInterval(()=>window.location.reload(),60000);
</script>
</body>
</html>
"""

def get_tag_name(premium):
    if premium <0:
        return ("折价","tag-discount")
    elif premium <3:
        return ("较理想关注区间","tag-good")
    elif premium <5:
        return ("可以观察","tag-observe")
    else:
        return ("未触发","tag-none")

@app.route("/")
def index():
    return render_template_string(HTML_TEMPLATE, data=latest_data, logs=reversed(log_buffer), get_tag_name=get_tag_name)

if __name__=="__main__":
    t = threading.Thread(target=monitor_loop,daemon=True)
    t.start()
    app.run(host="0.0.0.0",port=8000)
