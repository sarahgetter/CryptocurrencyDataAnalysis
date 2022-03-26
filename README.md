#CrytocurrencyDataAnalysis #TimeScaleDBPractice

--Schema for cryptocurrency analysis
DROP TABLE IF EXISTS "currency_info";
CREATE TABLE "currency_info"(
   currency_code   VARCHAR (10),
   currency        TEXT
);

--Schema for btc_prices table
DROP TABLE IF EXISTS "btc_prices";
CREATE TABLE "btc_prices"(
   time            TIMESTAMP WITH TIME ZONE NOT NULL,
   opening_price   DOUBLE PRECISION,
   highest_price   DOUBLE PRECISION,
   lowest_price    DOUBLE PRECISION,
   closing_price   DOUBLE PRECISION,
   volume_btc      DOUBLE PRECISION,
   volume_currency DOUBLE PRECISION,
   currency_code   VARCHAR (10)
);

--Schema for crypto_prices table
DROP TABLE IF EXISTS "crypto_prices";
CREATE TABLE "crypto_prices"(
   time            TIMESTAMP WITH TIME ZONE NOT NULL,
   opening_price   DOUBLE PRECISION,
   highest_price   DOUBLE PRECISION,
   lowest_price    DOUBLE PRECISION,
   closing_price   DOUBLE PRECISION,
   volume_crypto   DOUBLE PRECISION,
   volume_btc      DOUBLE PRECISION,
   currency_code   VARCHAR (10)
);

--Schema for eth_prices table
DROP TABLE IF EXISTS "eth_prices";
CREATE TABLE "eth_prices"(
   time            TIMESTAMP WITH TIME ZONE NOT NULL,
   opening_price   DOUBLE PRECISION,
   highest_price   DOUBLE PRECISION,
   lowest_price    DOUBLE PRECISION,
   closing_price   DOUBLE PRECISION,
   volume_eth      DOUBLE PRECISION,
   volume_currency DOUBLE PRECISION,
   currency_code   VARCHAR (10)
);

--Timescale specific statements to create hypertables for better performance
SELECT create_hypertable('btc_prices', 'time');
SELECT create_hypertable('eth_prices', 'time');
SELECT create_hypertable('crypto_prices', 'time');

#####################################################################
#1. Import library and setup API key
#####################################################################
import requests
import json
import csv
from datetime import datetime

apikey = 'YOUR_CRYPTO_COMPARE_API_KEY'
#attach to end of URLstring
url_api_part = '&api_key=' + apikey

#####################################################################
#2. Populate list of all coin names
#####################################################################
#URL to get a list of coins from cryptocompare API
URLcoinslist = 'https://min-api.cryptocompare.com/data/all/coinlist'

#Get list of cryptos with their symbols
res1 = requests.get(URLcoinslist)
res1_json = res1.json()
data1 = res1_json['Data']
symbol_array = []
cryptoDict = dict(data1)

#write to CSV
with open('coin_names.csv', mode = 'w') as test_file:
   test_file_writer = csv.writer(test_file,
                                 delimiter = ',',
                                 quotechar = '"',
                                 quoting=csv.QUOTE_MINIMAL)
   for coin in cryptoDict.values():
     if day.get('time') == None:
       continue # skip this item
       name = coin['Name']
       symbol = coin['Symbol']
       symbol_array.append(symbol)
       coin_name = coin['CoinName']
       full_name = coin['FullName']
       entry = [symbol, coin_name]
       test_file_writer.writerow(entry)
print('Done getting crypto names and symbols. See coin_names.csv for result')

#####################################################################
#3. Populate historical price for each crypto in BTC
#####################################################################
#Note: this part might take a while to run since we're populating data for 4k+ coins
#counter variable for progress made
progress = 0
num_cryptos = str(len(symbol_array))
for symbol in symbol_array:
   # get data for that currency
   URL = 'https://min-api.cryptocompare.com/data/histoday?fsym=' +
         symbol +
         '&tsym=BTC&allData=true' +
         url_api_part
   res = requests.get(URL)
   res_json = res.json()
   data = res_json['Data']
   # write required fields into csv
   with open('crypto_prices.csv', mode = 'a') as test_file:
       test_file_writer = csv.writer(test_file,
                                     delimiter = ',',
                                     quotechar = '"',
                                     quoting=csv.QUOTE_MINIMAL)
       for day in data:
           rawts = day['time']
           ts = datetime.utcfromtimestamp(rawts).strftime('%Y-%m-%d %H:%M:%S')
           o = day['open']
           h = day['high']
           l = day['low']
           c = day['close']
           vfrom = day['volumefrom']
           vto = day['volumeto']
           entry = [ts, o, h, l, c, vfrom, vto, symbol]
           test_file_writer.writerow(entry)
   progress = progress + 1
   print('Processed ' + str(symbol))
   print(str(progress) + ' currencies out of ' +  num_cryptos + ' written to csv')
print('Done getting price data for all coins. See crypto_prices.csv for result')

#####################################################################
#4. Populate BTC prices in different fiat currencies
#####################################################################
# List of fiat currencies we want to query
# You can expand this list, but CryptoCompare does not have
# a comprehensive fiat list on their site
fiatList = ['AUD', 'CAD', 'CNY', 'EUR', 'GBP', 'GOLD', 'HKD',
'ILS', 'INR', 'JPY', 'KRW', 'PLN', 'RUB', 'SGD', 'UAH', 'USD', 'ZAR']

#counter variable for progress made
progress2 = 0
for fiat in fiatList:
   # get data for bitcoin price in that fiat
   URL = 'https://min-api.cryptocompare.com/data/histoday?fsym=BTC&tsym=' +
         fiat +
         '&allData=true' +
         url_api_part
   res = requests.get(URL)
   res_json = res.json()
   data = res_json['Data']
   # write required fields into csv
   with open('btc_prices.csv', mode = 'a') as test_file:
       test_file_writer = csv.writer(test_file,
                                     delimiter = ',',
                                     quotechar = '"',
                                     quoting=csv.QUOTE_MINIMAL)
       for day in data:
           rawts = day['time']
           ts = datetime.utcfromtimestamp(rawts).strftime('%Y-%m-%d %H:%M:%S')
           o = day['open']
           h = day['high']
           l = day['low']
           c = day['close']
           vfrom = day['volumefrom']
           vto = day['volumeto']
           entry = [ts, o, h, l, c, vfrom, vto, fiat]
           test_file_writer.writerow(entry)
   progress2 = progress2 + 1
   print('processed ' + str(fiat))
   print(str(progress2) + ' currencies out of  17 written')
print('Done getting price data for btc. See btc_prices.csv for result')

#####################################################################
#5. Populate ETH prices in different fiat currencies
#####################################################################
#counter variable for progress made
progress3 = 0
for fiat in fiatList:
   # get data for bitcoin price in that fiat
   URL = 'https://min-api.cryptocompare.com/data/histoday?fsym=ETH&tsym=' +
         fiat +
         '&allData=true' +
         url_api_part
   res = requests.get(URL)
   res_json = res.json()
   data = res_json['Data']
   # write required fields into csv
   with open('eth_prices.csv', mode = 'a') as test_file:
       test_file_writer = csv.writer(test_file,
                                     delimiter = ',',
                                     quotechar = '"',
                                     quoting=csv.QUOTE_MINIMAL)
       for day in data:
           rawts = day['time']
           ts = datetime.utcfromtimestamp(rawts).strftime('%Y-%m-%d %H:%M:%S')
           o = day['open']
           h = day['high']
           l = day['low']
           c = day['close']
           vfrom = day['volumefrom']
           vto = day['volumeto']
           entry = [ts, o, h, l, c, vfrom, vto, fiat]
           test_file_writer.writerow(entry)
   progress3 = progress3 + 1
   print('processed ' + str(fiat))
   print(str(progress3) + ' currencies out of  17 written')
print('Done getting price data for eth. See eth_prices.csv for result')

python crypto_data_extraction.py

#psql -x "postgres://tsdbadmin:{YOUR_PASSWORD_HERE}@{YOUR_HOSTNAME_HERE}:{YOUR_PORT_HERE}/defaultdb?sslmode=require"

#CREATE DATABASE crypto_data;
#\c crypto_data
#CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;

#psql -x "postgres://tsdbadmin:{YOUR_PASSWORD_HERE}@{|YOUR_HOSTNAME_HERE}:{YOUR_PORT_HERE}/crypto_data?sslmode=require" < schema.sql
