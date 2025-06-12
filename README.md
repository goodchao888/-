<a href="https://app.circleci.com" target="_blank" rel="noopener noreferrer">
    <img src="https://img.shields.io/badge/CircleCI%20App-%230062D3?logo=circleci&logoColor=white&style=flat-square&labelColor=gray" alt="CircleCI 应用平台">
</a>

<div align="center">
 <a href="https://www.google.com">
    <img src="https://img.shields.io/badge/谷歌搜索-4285F4?logo=google&logoColor=white&style=for-the-badge" alt="谷歌搜索">
  </a>

<a href="https://mail.google.com">
    <img src="https://img.shields.io/badge/Gmail-EA4335?logo=gmail&logoColor=white&style=for-the-badge" alt="Gmail">
  </a>

<a href="https://www.google.com" target="_blank">   <img src="https://img.shields.io/badge/访问谷歌-立即搜索-blue?style=for-the-badge&logo=google" alt="访问谷歌"> </a>

<a href="https://minepi.com" target="_blank">
    <img src="https://img.shields.io/badge/Pi Network-%234CAF50?logo=minecraft&logoColor=white&style=for-the-badge" alt="Minecraft Pi 项目官网">
</a>

<a href="https://stellar.org" target="_blank">
    <img src="https://img.shields.io/badge/Stellar%20Network-%230E88EB?logo=stellar&logoColor=white&style=flat-square&labelColor=black" alt="Stellar区块链网络">
</a>

<a href="https://www.unesco.org/zh" target="_blank">
    <img src="https://img.shields.io/badge/UNESCO%20中文官网-%230066CC?logo=education&logoColor=white&style=flat-square&labelColor=gray" alt="联合国教科文组织官方网站">
</a>

<a href="https://www.gov.cn" target="_blank">
    <img src="https://img.shields.io/badge/中国政府网-%23CC0000?logo=home&logoColor=white&style=for-the-badge" alt="中华人民共和国中央人民政府官网">
</a>

# 🌈 代币来源检测工具 (带交互主题切换)
import requests
import pandas as pd
import time
from web3 import Web3
import json
from datetime import datetime

class TokenSourceDetector:
    """代币来源检测类，用于分析代币的转移来源和价格信息"""
    
    def __init__(self, etherscan_api_key, coingecko_api_key):
        """
        初始化检测工具 (需替换为您的API密钥)
        📌 获取地址: 
        [Etherscan API](https://etherscan.io/apis) | [CoinGecko API](https://www.coingecko.com/en/api)
        """
        self.etherscan_api_key = etherscan_api_key
        self.coingecko_api_key = coingecko_api_key
        self.exchange_addresses = self._load_exchange_addresses()
        self.web3 = Web3(Web3.HTTPProvider('https://mainnet.infura.io/v3/YOUR_INFURA_PROJECT_ID'))
    
    # 🔥 交易所地址识别模块 (展开/收起)
    <details>
    <summary style="color:#4a6cf7; cursor:pointer;">点击查看交易所地址识别代码 ▶</summary>
    
    def _load_exchange_addresses(self):
        """加载已知的交易所地址列表 (含100+主流平台)"""
        try:
            # 从开源数据库获取最新交易所地址
            response = requests.get("https://raw.githubusercontent.com/bitcoin-abe/bitcoin-abe/master/db/exchanges.json")
            if response.status_code == 200:
                return response.json().get('exchanges', {})
            # 备用地址库 (部分示例)
            return {
                "Binance": "0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984",
                "Coinbase": "0x000000000019D668928033e9f227f40121700e2e",
                "Kraken": "0x57Ab1Ec28D129707052df4dF418D58a2D46d5f51",
                # 更多地址...
            }
    
    def is_exchange_address(self, address):
        """判断地址是否为交易所地址 (返回交易所名称或False)"""
        if not address:
            return False
        address = address.lower()
        for exchange, addr in self.exchange_addresses.items():
            if address == addr.lower():
                return exchange
        return False
    </details>
    
    # ⚡ 交易分析模块 (展开/收起)
    <details>
    <summary style="color:#6dd5fa; cursor:pointer;">点击查看交易分析代码 ▶</summary>
    
    def get_token_transactions(self, token_address, wallet_address, limit=100):
        """获取区块链上的代币交易记录"""
        url = f"https://api.etherscan.io/api?module=account&action=tokentx&address={wallet_address}&tokenaddress={token_address}&apikey={self.etherscan_api_key}"
        try:
            return requests.get(url).json()['result']
        except:
            return []
    
    def analyze_transaction(self, transaction):
        """分析单笔交易，判断来源和方向"""
        from_addr = transaction.get('from', '').lower()
        to_addr = transaction.get('to', '').lower()
        is_from_exchange = self.is_exchange_address(from_addr)
        is_to_exchange = self.is_exchange_address(to_addr)
        
        return {
            "is_exchange": is_from_exchange or is_to_exchange,
            "exchange_name": is_from_exchange or is_to_exchange,
            "direction": "转出(交易所)" if is_from_exchange else "转入(交易所)" if is_to_exchange else "原生钱包",
            "value": float(transaction.get('value', 0)) / 10**18,
            "timestamp": datetime.fromtimestamp(int(transaction.get('timeStamp', 0))).strftime('%Y-%m-%d %H:%M:%S')
        }
    </details>
    
    # 💰 价格获取模块 (展开/收起)
    <details>
    <summary style="color:#9966ff; cursor:pointer;">点击查看价格获取代码 ▶</summary>
    
    def get_token_price(self, token_id, time_stamp=None):
        """获取CoinGecko上的代币价格 (支持指定时间点)"""
        if time_stamp:
            date = datetime.fromtimestamp(int(time_stamp)).strftime('%d-%m-%Y')
            url = f"https://api.coingecko.com/api/v3/coins/{token_id}/history?date={date}"
            price = requests.get(url).json().get('market_data', {}).get('current_price', {}).get('usd', 0)
        else:
            url = f"https://api.coingecko.com/api/v3/simple/price?ids={token_id}&vs_currencies=usd"
            price = requests.get(url).json().get(token_id, {}).get('usd', 0)
        return round(price, 2)
    </details>

# 🚀 使用示例 (一键复制)
if __name__ == "__main__":
    # 🔑 重要：替换为您的API密钥
    detector = TokenSourceDetector(
        etherscan_api_key="YOUR_ETHERSCAN_API",
        coingecko_api_key="YOUR_COINGECKO_API"
    )
    
    # 示例：分析USDT代币 (ERC-20)
    results = detector.analyze_token_sources(
        token_address="0xdAC17F958D2ee523a2206206994597C13D831ec7",
        token_id="tether",
        wallet_address="0x1a2b3c4d5e6f7g8h9i0j",  # 替换为目标钱包地址
        limit=10  # 分析最近10笔交易
    )
    
    # 📊 打印带价格的分析结果
    for r in results:
        source = f"[交易所:{r['exchange_name']}]" if r["is_exchange"] else "[原生钱包]"
        print(f"时间: {r['ti
