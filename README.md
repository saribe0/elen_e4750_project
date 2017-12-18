# Parallelized NLP Market Prediction
Stock market prediction through parallel processing of news stories and basic machine learning.

## Abstract
Many consider the stock market to be efficient, however, many of these efficiencies are based on numerical data (ie. past values and financials). There is, however, a wealth of textual data which influences investors’ decisions and may lead to inefficiencies as it is hard to analyze quickly. This project explores how basic Natural Language Processing (NLP) algorithms might be parallelized to take advantage of these inefficiencies. It explores the challenges of memory management and limited string operations in parallelized NLP as well as its feasibility. Finally, we present a possible solution to these considerations and show that parallelization has the potential to speed up NLP by 1.5 to 10 times.

## File Structure
```
./
./data/                                                 <= Folder containing all the data for this project
./data/articles/*                                       <= Folders for each day of loaded articles
./data/articles/stock_market_prediction-10-28-2017/*    <= Articles for each stock by ticker stored in text files for the day specified by the folder
./data/stock_price_data.txt                             <= Text file containing stock price data. This file serves as the database storage on disk for all the pulled stock prices.
./data/(word_weight_data_opt1.txt)                      <= Not initially in the folder but will be created when the update command is run for the first time. 
                                                           This file contains the learned word weights for the basic weighting scheme.
./data/(word_weight_data_opt2.txt)                      <= Not initially in the folder but will be created when the update command is run for the first time. 
                                                           This file contains the learned word weights for the Bayesian weighting scheme.
./output/*                                              <= Created when the first prediction is made and stores every prediction made by the program for the method and day indicated by the name of the text file.
./gpu_device_query.out                                  <= Output of a device query for the device testing was completed on.
./prediction_accuracy.png                               <= Generated graph of prediction acuracy from late October to early December based on the data available in the repository.
./README.md                                             <= This file.
./stock_market_prediction.py                            <= Python file containing all the code for the project. How to run and test is listed below. Examine comments and function names for clarity.                                                   ./E4750_2017Fall_PSMP_srb2208.report.pdf                <= Report for this project in pdf format.                           
```
## Project Sub-Modules
To accomplish the objectives listed in the abstract, this project implemnets 2 basic word weighting schemes which are used in 7 different prediction methdos. To do this, we split the project into four basic sub-modules. They are: Pull Articles, Predict Stock Movements, Pull Stock Prices, and Update Word Weights. Each one is explained a bit more in the following subsections. See the how to run section on how to execute these modules.
### Pull Articles
This module uses Python's request library to scrape Google News to retrieve relevent financial articles about specific stocks. The Beautiful Soup library is used to parse through the returned HTML and the articles are stored in text files in the `./data/articles/` folder.
### Predict Stock Movements
This module goes through the articles generated by the pull articles module and the word weights generated by the update word weights module to predict stock movements. Predictions can be made using a basic weighting scheme (opt1) and a Bayesian classifier (opt2). This module can only be run after the other three have been run at least once and word weights have been generated for the desired weighting scheme. The outputs of this module are stored in the `./output/` folder. This module is parallelized.
### Pull Stock Prices
Similar to the pull aticles module, this module uses Python's requests library to scrape Google Finance and retrieve the open and close prices for a stock for a specific day. It saves them all in the text file: `./data/stock_price_data.txt`
### Update Word Weights
This update word weights module goes through articles pulled for a stock and the price movements of that stock and updates the word weights based on the price movements. There are two weighting options implemented: a basic one, and a Bayesian classification one. The Basic one keeps track of upward stock occurrences and total occurrences. The Bayesian one keeps track of upward stock occurrences and downward stock occurrences. These are kept individually for each word. Running this module for the first time will create a word weights database. It can be run for both weighting types using opt1 and opt2 as explained in How to Run. This module requires that the pull articles and pull stock prices modules have already been run on the day attempting to update weights for. The module's databases are saved to the following files:
```
./data/word_weight_data_opt1.txt
./data/word_weight_data_opt2.txt
```
This module is parallelized.

## How To Run
### GPU vs. CPU
This code is designed to be run on a general purpose graphics processing unit or a normal computer (CPU). If you wish to use a GPU, timing analysis and output comparisons will also be run so it will take considerably longer. To indicate which type of operation you would like to use, set the flag at the top of the file. Use `GPU = True` to enable the GPU code and `GPU = False` to run on a CPU. Some commands such as pulling articles and stock prices are not parallelized due to network constraints and cannot be run on GPU (or at least the class server we are testing on due to missing librarys). A library of articles and a database of stock prices is included so these functions are not necessary for testing.

### A Full Test of Parallelizable Aspects
The parallelizable aspects of this project can be tested on the class GPU by doing the following. First, clone the git repository to the server. It contains a collection of news articles and a database of stock prices that can be used in testing. The folder structure is explained above. The root of the directory contains `stock_market_prediction.py`. This is the file that contains all the functions and code. 

#### Train Two Models Over 5 Days
The downloaded repository does not have any models trained so the first step is to train a model for both the basic weighting scheme and the Bayesian weighting scheme. Running the following commands from the root directory will train the model over 5 days. You can adjust the end date to train over more days, however, it each day added will take longer as everything is being calculated by the GPU and CPU for comparison. Word weights for both weighting schemes will be updated using the GPU. The first command is for the basic word weights, the second is for the Bayesian ones.

```
sbatch --gres=gpu:1 --time=20 --wrap="python stock_market_prediction.py -u -b 11-1-2017 -e 11-7-2017"
sbatch --gres=gpu:1 --time=20 --wrap="python stock_market_prediction.py -u -b 11-1-2017 -e 11-7-2017 -o opt2"
```
The `-u` in the commands is for update word weights, the `-b` indicates the start day and `-e` indicates the end day. The `-o` indicates the type of weight to be updated. The default is `opt1` for the basic weighting scheme which is why it is left out of the first command.

#### Make A Prediction
Once trained, a prediction can be made. A prediction for the next day using both weighting types can be done through the following commands. The first is for making predictions with the basic weighting scheme and the second is for making predictions using the Bayesian classifier.
```
sbatch --gres=gpu:1 --time=20 --wrap="python stock_market_prediction.py -p -d 11-8-2017"
sbatch --gres=gpu:1 --time=20 --wrap="python stock_market_prediction.py -p -d 11-8-2017 -o opt2"
```
The `-p` indicates to make a prediction, the `-d` is to specify the day to make the prediction. The update command from the previous section can also use the `-d` option to update weights for a specific day. Once again `-o` is used to signal the Bayesian classifier in the second command.

#### Checking Output
By "cat"-ing the slurm files, you will be able to see the output of the commands. The update commands will just list the functions being run, the accuracy between of the GPU and the time difference. The listed speedup will likely be fairly low due to it starting from an uninitialized database. The report goes into more depth on why this is the case.The output of the prediction commands will show the predictions for November 8th for each of the 20 stocks with some stats about the prediction. At the bottom of both outputs will be the accuracy and timing. The output of the prediction using the basic weights will also include weight analysis accuracy and timing. The Bayesian classifier prediction does not analyze the weights.
#### Expected Output of Test
After running for a short while, we get the following output:
##### For the first update function from "Train Two Models Over 5 Days":
The expected output of the slurm file for this command is:
```
WARNING:root:- Could not load word weights, Error: [Errno 2] No such file or directory: './data/word_weight_data_opt1.txt'
ERROR:root:Could not find articles to load for: 11-4-2017
ERROR:root:Could not find articles to load for: 11-5-2017
WARNING:root:- Could not load word weights, Error: [Errno 2] No such file or directory: './data/word_weight_data_opt1.txt'
ERROR:root:Could not find articles to load for: 11-4-2017
ERROR:root:Could not find articles to load for: 11-5-2017
Loading stock prices
Loading word weights
Loading articles
Updating word weights
Loading articles
Updating word weights
Loading articles
Updating word weights
Loading articles
Updating word weights
Loading articles
Updating word weights
Loading word weights
Loading articles
Updating word weights with gpu
Loading articles
Updating word weights with gpu
Loading articles
Updating word weights with gpu
Loading articles
Updating word weights with gpu
Loading articles
Updating word weights with gpu
Saving word weights

Update Avg Percent Difference: 0 %

Update Speedup: Kernel = 4.07231672133, Function = 2.44689113768

Done
```
The speedup is, of course, dependent on the exact run. The values should be minimal because we are starting from an uninitialized database, however, please remember that these speedups include the extra processing required to compare the CPU and GPU that wouldn't normally be used. This is explained more in the report. When checking the `./data/` folder, the following file should have been generated: `./data/word_weight_data_opt1.txt`

##### For the second update function from "Train Two Models Over 5 Days":
The expected output of the slurm file for this command is:
```
WARNING:root:- Could not load word weights, Error: [Errno 2] No such file or directory: './data/word_weight_data_opt2.txt'
ERROR:root:Could not find articles to load for: 11-4-2017
ERROR:root:Could not find articles to load for: 11-5-2017
WARNING:root:- Could not load word weights, Error: [Errno 2] No such file or directory: './data/word_weight_data_opt2.txt'
ERROR:root:Could not find articles to load for: 11-4-2017
ERROR:root:Could not find articles to load for: 11-5-2017
Loading stock prices
Loading word weights
Loading articles
Updating word weights
Loading articles
Updating word weights
Loading articles
Updating word weights
Loading articles
Updating word weights
Loading articles
Updating word weights
Loading word weights
Loading articles
Updating word weights with gpu
Loading articles
Updating word weights with gpu
Loading articles
Updating word weights with gpu
Loading articles
Updating word weights with gpu
Loading articles
Updating word weights with gpu
Saving word weights

Update Avg Percent Difference: 0 %

Update Speedup: Kernel = 4.06432365745, Function = 2.45313616393

Done
```
The speedup is, of course, dependent on the exact run, The values should be minimal because we are starting from an uninitialized database, however, please remember that these speedups include the extra processing required to compare the CPU and GPU that wouldn't normally be used. This is explained more in the report. When checking the `./data/` folder, the following file should have been generated: `./data/word_weight_data_opt2.txt`

##### For the first prediction function from "Make A Prediction":
Since this commands output will be quite large, we will only list the first few stocks and then skip to the end of the file. The expected output for the first five stocks and accuracy/speedup stats are:
```
Loading word weights
Loading articles
Analyzing weights
Analyzing weights with gpu

PREDICTIONS FOR METHOD 1 BASED ON:
('\t- AVG: ', 0.40382395628237688)
('\t- STD: ', 0.3470834424144252)
('RATING FOR: ', 'amzn')
('\t- STD ABOVE MEAN: ', 0.2057480781981616)
('\t- RAW VAL RATING: ', 0.47523570753354716)
('\t- PROBABILITY IS: ', 0.58150614953004254)
('\t- CORRESPONDS TO: ', 'undecided')
('RATING FOR: ', 'amat')
('\t- STD ABOVE MEAN: ', 0.20417908086945469)
('\t- RAW VAL RATING: ', 0.47469113453956052)
('\t- PROBABILITY IS: ', 0.58089322097206253)
('\t- CORRESPONDS TO: ', 'undecided')
('RATING FOR: ', 'agn')
('\t- STD ABOVE MEAN: ', 0.084322376126346674)
('\t- RAW VAL RATING: ', 0.43309085686087323)
('\t- PROBABILITY IS: ', 0.53359993890990809)
('\t- CORRESPONDS TO: ', 'undecided')
('RATING FOR: ', 'goog')
('\t- STD ABOVE MEAN: ', 0.26966923113074864)
('\t- RAW VAL RATING: ', 0.4974216813364884)
('\t- PROBABILITY IS: ', 0.60629263308516901)
('\t- CORRESPONDS TO: ', 'undecided')
('RATING FOR: ', 'hd')
('\t- STD ABOVE MEAN: ', 0.11119865472510113)
('\t- RAW VAL RATING: ', 0.44241916815621807)
('\t- PROBABILITY IS: ', 0.5442705908039871)
('\t- CORRESPONDS TO: ', 'undecided')

/********
Many other predictions
 ********/
 
Analysis Avg Percent Difference: 3.93541291826e-07 %
Prediction Avg Percent Difference: 0.010266189542 %

Analysis Speedup: Kernel = 40.6981240092, Function = 0.630166790988
Prediction Speedup: Kernel = 29.1048975375, Function = 5.6398644511

Done
```
The speedup and exact accuracy is, of course, dependent on the exact run. Please remember that these speedups include the extra processing required to compare the CPU and GPU that wouldn't normally be used - See the report for averaged speedups without this processing overhead. In the case of analyzing stock weights, this has a significant affect on the functional speedup which is why it is less than 1. This run should create the following files which also has the full output for their respective prediction methods:
```
prediction-11-8-2017.txt
prediction2-11-8-2017.txt
prediction3-11-8-2017.txt
prediction4-11-8-2017.txt
prediction5-11-8-2017.txt
prediction6-11-8-2017.txt
```
##### For the second prediction function from "Make A Prediction":
Since this commands output will be quite large, we will only list the first few stocks and then skip to the end of the file. The expected output for the first five stocks and accuracy/speedup stats are:
```
Loading word weights
Loading articles

BAYES PREDICTIONS 
('RATING FOR: ', 'amzn')
('\t- UP RATING IS  : ', -32496.228423222165)
('\t- DOWN RATING IS: ', -32750.06874186211)
('\t- CORRESPONDS TO: ', 'buy')
('RATING FOR: ', 'amat')
('\t- UP RATING IS  : ', -34058.5754374649)
('\t- DOWN RATING IS: ', -34410.98348404315)
('\t- CORRESPONDS TO: ', 'buy')
('RATING FOR: ', 'agn')
('\t- UP RATING IS  : ', -34496.254208406026)
('\t- DOWN RATING IS: ', -33808.811410338014)
('\t- CORRESPONDS TO: ', 'sell')
('RATING FOR: ', 'goog')
('\t- UP RATING IS  : ', -32966.81247995143)
('\t- DOWN RATING IS: ', -34044.891535898605)
('\t- CORRESPONDS TO: ', 'buy')
('RATING FOR: ', 'hd')
('\t- UP RATING IS  : ', -34974.7112359665)
('\t- DOWN RATING IS: ', -34367.416998004905)
('\t- CORRESPONDS TO: ', 'sell')

/********
Many other predictions
 ********/
 
Prediction Avg Percent Difference: 0.0316892841531 %

Prediction Speedup: Kernel = 36.2657635688, Function = 5.72629179833

Done
```
The speedup and exact accuracy is, of course, dependent on the exact run. Please remember that these speedups include the extra processing required to compare the CPU and GPU that wouldn't normally be used. Also note that "buy" is equivilent to an "up" prediction as discussed in the report and "sell" is equivilent to a down prediction. This run should create the following file which also has the full output: `prediction7-11-8-2017.txt`

#### Next Testing Steps
If more tests are desired, you can continue to run the update and predict commands on different days. If you run them on days without stock price data or articles, you will recieve an error. You can determine which days have downloaded articles by examining the ./data/articles/ folder. Each day there is an article has a price in the database except December 5th which has, so far, only been used for predicting. While running subsequent update commands, the speedups should be higher due to already having the word weight databases initialized.

### Available Commands
To run any of the parallelizable ones with the GPU, they must be wrapped in an sbatch and python must be specified:
```
sbatch --gres=gpu:1 --time=20 --wrap="python stock_market_prediction.py <options>"
```
Any of the commands that say "GPU Possible" must have `GPU = True` at the top of the file to use the GPU. Other commands should not be run using the above wrapper. When switching between types, make sure to check the `GPU =`. Many of these commands may use python libraries that are not on the course server. For this project we focus on the parallelizable aspects which are all runable.

#### PULLING ARTICLES:

For the current day:
```
./stock_market_prediction.py -a
```

Supported Options:
```
none
```

#### PREDICTIONS (GPU Possible):
(articles for the requested day must already be pulled)

For the current day with basic weights:
```
./stock_market_prediction.py -p
```

Supported Options:
```
-o <weight options>
-d <specified date>
```

#### PULLING STOCK PRICES:

For the current day:
```
./stock_market_prediction.py -s
```

Supported Options:
```
none
```

#### UPDATE WEIGHTS (GPU Possible):
(articles and stock prices for the requested days must already be pulled)

For the update basic weights for the current day:
```
./stock_market_prediction.py -u
```

Supported Options:
```
-o <weight options>
-d <specified date>
-b <start date>
-e <end date>
```
If a start date is given without an end date, the end date will default to the current date. The specified date option does not work with the start or end date options. If there are dates within the date range (from -b date to -e date or current date) that do not have articles or stock data (such as weekends), they are skipped.

#### HELP:

To get this list printed:
```
./stock_market_prediction.py -h
```

#### WEIGHT ANALYSIS (helper function that just prints weight statistics):
(must have weights calculated for the standard weight option)

Print the weight statistics:
```
./stock_market_prediction.py -z
```

#### DETERMINE ACCURACY (helper function for analyzing predictions over time):

Determine the accuracy of any predictions in the ./output/ folder:
```
./stock_market_prediction.py -v
```

This command also generates the graph of accuracy relative to the days the predictions were made.



