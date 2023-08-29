# Prediction-of-Solar-Irradiance
{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Data Description"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "These datasets are meteorological data from the HI-SEAS weather station from four months (September through December 2016) between Mission IV and Mission V.\n",
    "\n",
    "For each dataset, the fields are:\n",
    "\n",
    "A row number (1-n) useful in sorting this export's results\n",
    "The UNIX time_t date (seconds since Jan 1, 1970). Useful in sorting this export's results with other export's results\n",
    "The date in yyyy-mm-dd format\n",
    "The local time of day in hh:mm:ss 24-hour format\n",
    "The numeric data, if any (may be an empty string)\n",
    "The text data, if any (may be an empty string)\n",
    "\n",
    "The units of each dataset are:\n",
    "\n",
    "- Solar radiation: watts per meter^2\n",
    "- Temperature: degrees Fahrenheit\n",
    "- Humidity: percent\n",
    "- Barometric pressure: Hg\n",
    "- Wind direction: degrees\n",
    "- Wind speed: miles per hour\n",
    "- Sunrise/sunset: Hawaii time\n",
    "\n",
    "Link: https://www.kaggle.com/datasets/dronio/SolarEnergy"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Table of Content"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "1. Importing Libraries\n",
    "2. Loading Data\n",
    "3. Data Wrangling\n",
    "4. Feature Selection using Correlation Matrix\n",
    "5. Feature Selection using SelectKBest Method\n",
    "6. Feature Selection using Extra Tree Classifier\n",
    "7. Feature Engineering with BoxCox, Log, Min-Max and Standard transformation\n",
    "8. Preparing data - Standardisation and Splitting\n",
    "9. Prediction with XGBoost\n",
    "10. Using MultiLayer Perceptron for prediction"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Importing Libraries"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 338,
   "metadata": {},
   "outputs": [],
   "source": [
    "import warnings\n",
    "warnings.filterwarnings(\"ignore\")\n",
    "import numpy as np\n",
    "from scipy import stats\n",
    "import pandas as pd\n",
    "import matplotlib.pyplot as plt\n",
    "import seaborn as sns\n",
    "\n",
    "import re\n",
    "\n",
    "from sklearn.preprocessing import StandardScaler, MinMaxScaler\n",
    "\n",
    "from sklearn.ensemble import ExtraTreesClassifier\n",
    "from sklearn.feature_selection import SelectKBest\n",
    "from sklearn.feature_selection import chi2\n",
    "from sklearn.model_selection import train_test_split\n",
    "\n",
    "# !pip install xgboost\n",
    "import xgboost as xgb\n",
    "\n",
    "from tensorflow.keras.layers import Dense, Dropout, Activation\n",
    "from tensorflow.keras.optimizers import SGD, Adam\n",
    "from tensorflow.keras.models import Sequential\n",
    "from collections import Counter\n",
    "\n",
    "from sklearn.metrics import mean_squared_error, r2_score, mean_absolute_error"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Loading Data"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 41,
   "metadata": {},
   "outputs": [],
   "source": [
    "data = pd.read_csv(\"solar_irradiance data.csv\")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 42,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>UNIXTime</th>\n",
       "      <th>Data</th>\n",
       "      <th>Time</th>\n",
       "      <th>Radiation</th>\n",
       "      <th>Temperature</th>\n",
       "      <th>Pressure</th>\n",
       "      <th>Humidity</th>\n",
       "      <th>WindDirection(Degrees)</th>\n",
       "      <th>Speed</th>\n",
       "      <th>TimeSunRise</th>\n",
       "      <th>TimeSunSet</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>1475229326</td>\n",
       "      <td>9/29/2016 12:00:00 AM</td>\n",
       "      <td>23:55:26</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>59</td>\n",
       "      <td>177.39</td>\n",
       "      <td>5.62</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>1475229023</td>\n",
       "      <td>9/29/2016 12:00:00 AM</td>\n",
       "      <td>23:50:23</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>58</td>\n",
       "      <td>176.78</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>1475228726</td>\n",
       "      <td>9/29/2016 12:00:00 AM</td>\n",
       "      <td>23:45:26</td>\n",
       "      <td>1.23</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>57</td>\n",
       "      <td>158.75</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>1475228421</td>\n",
       "      <td>9/29/2016 12:00:00 AM</td>\n",
       "      <td>23:40:21</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>60</td>\n",
       "      <td>137.71</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>1475228124</td>\n",
       "      <td>9/29/2016 12:00:00 AM</td>\n",
       "      <td>23:35:24</td>\n",
       "      <td>1.17</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>62</td>\n",
       "      <td>104.95</td>\n",
       "      <td>5.62</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "     UNIXTime                   Data      Time  Radiation  Temperature  \\\n",
       "0  1475229326  9/29/2016 12:00:00 AM  23:55:26       1.21           48   \n",
       "1  1475229023  9/29/2016 12:00:00 AM  23:50:23       1.21           48   \n",
       "2  1475228726  9/29/2016 12:00:00 AM  23:45:26       1.23           48   \n",
       "3  1475228421  9/29/2016 12:00:00 AM  23:40:21       1.21           48   \n",
       "4  1475228124  9/29/2016 12:00:00 AM  23:35:24       1.17           48   \n",
       "\n",
       "   Pressure  Humidity  WindDirection(Degrees)  Speed TimeSunRise TimeSunSet  \n",
       "0     30.46        59                  177.39   5.62    06:13:00   18:13:00  \n",
       "1     30.46        58                  176.78   3.37    06:13:00   18:13:00  \n",
       "2     30.46        57                  158.75   3.37    06:13:00   18:13:00  \n",
       "3     30.46        60                  137.71   3.37    06:13:00   18:13:00  \n",
       "4     30.46        62                  104.95   5.62    06:13:00   18:13:00  "
      ]
     },
     "execution_count": 42,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "data.head(5)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 43,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "<class 'pandas.core.frame.DataFrame'>\n",
      "RangeIndex: 32686 entries, 0 to 32685\n",
      "Data columns (total 11 columns):\n",
      " #   Column                  Non-Null Count  Dtype  \n",
      "---  ------                  --------------  -----  \n",
      " 0   UNIXTime                32686 non-null  int64  \n",
      " 1   Data                    32686 non-null  object \n",
      " 2   Time                    32686 non-null  object \n",
      " 3   Radiation               32686 non-null  float64\n",
      " 4   Temperature             32686 non-null  int64  \n",
      " 5   Pressure                32686 non-null  float64\n",
      " 6   Humidity                32686 non-null  int64  \n",
      " 7   WindDirection(Degrees)  32686 non-null  float64\n",
      " 8   Speed                   32686 non-null  float64\n",
      " 9   TimeSunRise             32686 non-null  object \n",
      " 10  TimeSunSet              32686 non-null  object \n",
      "dtypes: float64(4), int64(3), object(4)\n",
      "memory usage: 2.7+ MB\n"
     ]
    }
   ],
   "source": [
    "data.info()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Data Wrangling"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 55,
   "metadata": {},
   "outputs": [],
   "source": [
    "df = data.copy()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 56,
   "metadata": {},
   "outputs": [],
   "source": [
    "# extract the date from the date_time format of the 'Data' parameter\n",
    "df['Data'] = df['Data'].apply(lambda x : x.split()[0])"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 57,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>UNIXTime</th>\n",
       "      <th>Data</th>\n",
       "      <th>Time</th>\n",
       "      <th>Radiation</th>\n",
       "      <th>Temperature</th>\n",
       "      <th>Pressure</th>\n",
       "      <th>Humidity</th>\n",
       "      <th>WindDirection(Degrees)</th>\n",
       "      <th>Speed</th>\n",
       "      <th>TimeSunRise</th>\n",
       "      <th>TimeSunSet</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>1475229326</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:55:26</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>59</td>\n",
       "      <td>177.39</td>\n",
       "      <td>5.62</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>1475229023</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:50:23</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>58</td>\n",
       "      <td>176.78</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>1475228726</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:45:26</td>\n",
       "      <td>1.23</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>57</td>\n",
       "      <td>158.75</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>1475228421</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:40:21</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>60</td>\n",
       "      <td>137.71</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>1475228124</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:35:24</td>\n",
       "      <td>1.17</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>62</td>\n",
       "      <td>104.95</td>\n",
       "      <td>5.62</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "     UNIXTime       Data      Time  Radiation  Temperature  Pressure  \\\n",
       "0  1475229326  9/29/2016  23:55:26       1.21           48     30.46   \n",
       "1  1475229023  9/29/2016  23:50:23       1.21           48     30.46   \n",
       "2  1475228726  9/29/2016  23:45:26       1.23           48     30.46   \n",
       "3  1475228421  9/29/2016  23:40:21       1.21           48     30.46   \n",
       "4  1475228124  9/29/2016  23:35:24       1.17           48     30.46   \n",
       "\n",
       "   Humidity  WindDirection(Degrees)  Speed TimeSunRise TimeSunSet  \n",
       "0        59                  177.39   5.62    06:13:00   18:13:00  \n",
       "1        58                  176.78   3.37    06:13:00   18:13:00  \n",
       "2        57                  158.75   3.37    06:13:00   18:13:00  \n",
       "3        60                  137.71   3.37    06:13:00   18:13:00  \n",
       "4        62                  104.95   5.62    06:13:00   18:13:00  "
      ]
     },
     "execution_count": 57,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df.head()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 58,
   "metadata": {},
   "outputs": [],
   "source": [
    "# extract the date time features from the given parameter using date time python methods\n",
    "df['Month'] = pd.to_datetime(df['Data']).dt.month\n",
    "df['Day'] = pd.to_datetime(df['Data']).dt.day\n",
    "df['Hour'] = pd.to_datetime(df['Time']).dt.hour\n",
    "df['Minute'] = pd.to_datetime(df['Time']).dt.minute\n",
    "df['Second'] = pd.to_datetime(df['Time']).dt.second"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 59,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>UNIXTime</th>\n",
       "      <th>Data</th>\n",
       "      <th>Time</th>\n",
       "      <th>Radiation</th>\n",
       "      <th>Temperature</th>\n",
       "      <th>Pressure</th>\n",
       "      <th>Humidity</th>\n",
       "      <th>WindDirection(Degrees)</th>\n",
       "      <th>Speed</th>\n",
       "      <th>TimeSunRise</th>\n",
       "      <th>TimeSunSet</th>\n",
       "      <th>Month</th>\n",
       "      <th>Day</th>\n",
       "      <th>Hour</th>\n",
       "      <th>Minute</th>\n",
       "      <th>Second</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>1475229326</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:55:26</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>59</td>\n",
       "      <td>177.39</td>\n",
       "      <td>5.62</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>55</td>\n",
       "      <td>26</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>1475229023</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:50:23</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>58</td>\n",
       "      <td>176.78</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>50</td>\n",
       "      <td>23</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>1475228726</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:45:26</td>\n",
       "      <td>1.23</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>57</td>\n",
       "      <td>158.75</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>45</td>\n",
       "      <td>26</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>1475228421</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:40:21</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>60</td>\n",
       "      <td>137.71</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>40</td>\n",
       "      <td>21</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>1475228124</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:35:24</td>\n",
       "      <td>1.17</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>62</td>\n",
       "      <td>104.95</td>\n",
       "      <td>5.62</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>35</td>\n",
       "      <td>24</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "     UNIXTime       Data      Time  Radiation  Temperature  Pressure  \\\n",
       "0  1475229326  9/29/2016  23:55:26       1.21           48     30.46   \n",
       "1  1475229023  9/29/2016  23:50:23       1.21           48     30.46   \n",
       "2  1475228726  9/29/2016  23:45:26       1.23           48     30.46   \n",
       "3  1475228421  9/29/2016  23:40:21       1.21           48     30.46   \n",
       "4  1475228124  9/29/2016  23:35:24       1.17           48     30.46   \n",
       "\n",
       "   Humidity  WindDirection(Degrees)  Speed TimeSunRise TimeSunSet  Month  Day  \\\n",
       "0        59                  177.39   5.62    06:13:00   18:13:00      9   29   \n",
       "1        58                  176.78   3.37    06:13:00   18:13:00      9   29   \n",
       "2        57                  158.75   3.37    06:13:00   18:13:00      9   29   \n",
       "3        60                  137.71   3.37    06:13:00   18:13:00      9   29   \n",
       "4        62                  104.95   5.62    06:13:00   18:13:00      9   29   \n",
       "\n",
       "   Hour  Minute  Second  \n",
       "0    23      55      26  \n",
       "1    23      50      23  \n",
       "2    23      45      26  \n",
       "3    23      40      21  \n",
       "4    23      35      24  "
      ]
     },
     "execution_count": 59,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df.head()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 60,
   "metadata": {},
   "outputs": [],
   "source": [
    "# extract the sunrise and sunset information using regular expression\n",
    "df['risehour'] = df['TimeSunRise'].apply(lambda x : re.search(r'^\\d+', x).group(0)).astype(int)\n",
    "df['riseminuter'] = df['TimeSunRise'].apply(lambda x : re.search(r'(?<=\\:)\\d+(?=\\:)', x).group(0)).astype(int)\n",
    "\n",
    "df['sethour'] = df['TimeSunSet'].apply(lambda x : re.search(r'^\\d+', x).group(0)).astype(int)\n",
    "df['setminute'] = df['TimeSunSet'].apply(lambda x : re.search(r'(?<=\\:)\\d+(?=\\:)', x).group(0)).astype(int)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 61,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>UNIXTime</th>\n",
       "      <th>Data</th>\n",
       "      <th>Time</th>\n",
       "      <th>Radiation</th>\n",
       "      <th>Temperature</th>\n",
       "      <th>Pressure</th>\n",
       "      <th>Humidity</th>\n",
       "      <th>WindDirection(Degrees)</th>\n",
       "      <th>Speed</th>\n",
       "      <th>TimeSunRise</th>\n",
       "      <th>TimeSunSet</th>\n",
       "      <th>Month</th>\n",
       "      <th>Day</th>\n",
       "      <th>Hour</th>\n",
       "      <th>Minute</th>\n",
       "      <th>Second</th>\n",
       "      <th>risehour</th>\n",
       "      <th>riseminuter</th>\n",
       "      <th>sethour</th>\n",
       "      <th>setminute</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>1475229326</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:55:26</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>59</td>\n",
       "      <td>177.39</td>\n",
       "      <td>5.62</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>55</td>\n",
       "      <td>26</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>1475229023</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:50:23</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>58</td>\n",
       "      <td>176.78</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>50</td>\n",
       "      <td>23</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>1475228726</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:45:26</td>\n",
       "      <td>1.23</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>57</td>\n",
       "      <td>158.75</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>45</td>\n",
       "      <td>26</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>1475228421</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:40:21</td>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>60</td>\n",
       "      <td>137.71</td>\n",
       "      <td>3.37</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>40</td>\n",
       "      <td>21</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>1475228124</td>\n",
       "      <td>9/29/2016</td>\n",
       "      <td>23:35:24</td>\n",
       "      <td>1.17</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>62</td>\n",
       "      <td>104.95</td>\n",
       "      <td>5.62</td>\n",
       "      <td>06:13:00</td>\n",
       "      <td>18:13:00</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>35</td>\n",
       "      <td>24</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "     UNIXTime       Data      Time  Radiation  Temperature  Pressure  \\\n",
       "0  1475229326  9/29/2016  23:55:26       1.21           48     30.46   \n",
       "1  1475229023  9/29/2016  23:50:23       1.21           48     30.46   \n",
       "2  1475228726  9/29/2016  23:45:26       1.23           48     30.46   \n",
       "3  1475228421  9/29/2016  23:40:21       1.21           48     30.46   \n",
       "4  1475228124  9/29/2016  23:35:24       1.17           48     30.46   \n",
       "\n",
       "   Humidity  WindDirection(Degrees)  Speed TimeSunRise TimeSunSet  Month  Day  \\\n",
       "0        59                  177.39   5.62    06:13:00   18:13:00      9   29   \n",
       "1        58                  176.78   3.37    06:13:00   18:13:00      9   29   \n",
       "2        57                  158.75   3.37    06:13:00   18:13:00      9   29   \n",
       "3        60                  137.71   3.37    06:13:00   18:13:00      9   29   \n",
       "4        62                  104.95   5.62    06:13:00   18:13:00      9   29   \n",
       "\n",
       "   Hour  Minute  Second  risehour  riseminuter  sethour  setminute  \n",
       "0    23      55      26         6           13       18         13  \n",
       "1    23      50      23         6           13       18         13  \n",
       "2    23      45      26         6           13       18         13  \n",
       "3    23      40      21         6           13       18         13  \n",
       "4    23      35      24         6           13       18         13  "
      ]
     },
     "execution_count": 61,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df.head()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 107,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "<class 'pandas.core.frame.DataFrame'>\n",
      "RangeIndex: 32686 entries, 0 to 32685\n",
      "Data columns (total 15 columns):\n",
      " #   Column                  Non-Null Count  Dtype  \n",
      "---  ------                  --------------  -----  \n",
      " 0   Radiation               32686 non-null  float64\n",
      " 1   Temperature             32686 non-null  int64  \n",
      " 2   Pressure                32686 non-null  float64\n",
      " 3   Humidity                32686 non-null  int64  \n",
      " 4   WindDirection(Degrees)  32686 non-null  float64\n",
      " 5   Speed                   32686 non-null  float64\n",
      " 6   Month                   32686 non-null  int64  \n",
      " 7   Day                     32686 non-null  int64  \n",
      " 8   Hour                    32686 non-null  int64  \n",
      " 9   Minute                  32686 non-null  int64  \n",
      " 10  Second                  32686 non-null  int64  \n",
      " 11  risehour                32686 non-null  int64  \n",
      " 12  riseminuter             32686 non-null  int64  \n",
      " 13  sethour                 32686 non-null  int64  \n",
      " 14  setminute               32686 non-null  int64  \n",
      "dtypes: float64(4), int64(11)\n",
      "memory usage: 3.7 MB\n"
     ]
    }
   ],
   "source": [
    "df.info()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 62,
   "metadata": {},
   "outputs": [],
   "source": [
    "# drop the parameters that are not required after extracting the relevant information\n",
    "df.drop(['UNIXTime', 'Data', 'Time', 'TimeSunRise', 'TimeSunSet'], axis = 1, inplace = True)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 63,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Radiation</th>\n",
       "      <th>Temperature</th>\n",
       "      <th>Pressure</th>\n",
       "      <th>Humidity</th>\n",
       "      <th>WindDirection(Degrees)</th>\n",
       "      <th>Speed</th>\n",
       "      <th>Month</th>\n",
       "      <th>Day</th>\n",
       "      <th>Hour</th>\n",
       "      <th>Minute</th>\n",
       "      <th>Second</th>\n",
       "      <th>risehour</th>\n",
       "      <th>riseminuter</th>\n",
       "      <th>sethour</th>\n",
       "      <th>setminute</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>59</td>\n",
       "      <td>177.39</td>\n",
       "      <td>5.62</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>55</td>\n",
       "      <td>26</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>58</td>\n",
       "      <td>176.78</td>\n",
       "      <td>3.37</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>50</td>\n",
       "      <td>23</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>1.23</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>57</td>\n",
       "      <td>158.75</td>\n",
       "      <td>3.37</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>45</td>\n",
       "      <td>26</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>60</td>\n",
       "      <td>137.71</td>\n",
       "      <td>3.37</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>40</td>\n",
       "      <td>21</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>1.17</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>62</td>\n",
       "      <td>104.95</td>\n",
       "      <td>5.62</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>35</td>\n",
       "      <td>24</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   Radiation  Temperature  Pressure  Humidity  WindDirection(Degrees)  Speed  \\\n",
       "0       1.21           48     30.46        59                  177.39   5.62   \n",
       "1       1.21           48     30.46        58                  176.78   3.37   \n",
       "2       1.23           48     30.46        57                  158.75   3.37   \n",
       "3       1.21           48     30.46        60                  137.71   3.37   \n",
       "4       1.17           48     30.46        62                  104.95   5.62   \n",
       "\n",
       "   Month  Day  Hour  Minute  Second  risehour  riseminuter  sethour  setminute  \n",
       "0      9   29    23      55      26         6           13       18         13  \n",
       "1      9   29    23      50      23         6           13       18         13  \n",
       "2      9   29    23      45      26         6           13       18         13  \n",
       "3      9   29    23      40      21         6           13       18         13  \n",
       "4      9   29    23      35      24         6           13       18         13  "
      ]
     },
     "execution_count": 63,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df.head()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 64,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "(32686, 15)"
      ]
     },
     "execution_count": 64,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# check of data dimensions\n",
    "df.shape"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 65,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0"
      ]
     },
     "execution_count": 65,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# checking for null values in the data\n",
    "df.isnull().sum().sum()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 102,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Radiation</th>\n",
       "      <th>Temperature</th>\n",
       "      <th>Pressure</th>\n",
       "      <th>Humidity</th>\n",
       "      <th>WindDirection(Degrees)</th>\n",
       "      <th>Speed</th>\n",
       "      <th>Month</th>\n",
       "      <th>Day</th>\n",
       "      <th>Hour</th>\n",
       "      <th>Minute</th>\n",
       "      <th>Second</th>\n",
       "      <th>risehour</th>\n",
       "      <th>riseminuter</th>\n",
       "      <th>sethour</th>\n",
       "      <th>setminute</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>59</td>\n",
       "      <td>177.39</td>\n",
       "      <td>5.62</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>55</td>\n",
       "      <td>26</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>58</td>\n",
       "      <td>176.78</td>\n",
       "      <td>3.37</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>50</td>\n",
       "      <td>23</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>1.23</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>57</td>\n",
       "      <td>158.75</td>\n",
       "      <td>3.37</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>45</td>\n",
       "      <td>26</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>1.21</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>60</td>\n",
       "      <td>137.71</td>\n",
       "      <td>3.37</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>40</td>\n",
       "      <td>21</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>1.17</td>\n",
       "      <td>48</td>\n",
       "      <td>30.46</td>\n",
       "      <td>62</td>\n",
       "      <td>104.95</td>\n",
       "      <td>5.62</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>35</td>\n",
       "      <td>24</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   Radiation  Temperature  Pressure  Humidity  WindDirection(Degrees)  Speed  \\\n",
       "0       1.21           48     30.46        59                  177.39   5.62   \n",
       "1       1.21           48     30.46        58                  176.78   3.37   \n",
       "2       1.23           48     30.46        57                  158.75   3.37   \n",
       "3       1.21           48     30.46        60                  137.71   3.37   \n",
       "4       1.17           48     30.46        62                  104.95   5.62   \n",
       "\n",
       "   Month  Day  Hour  Minute  Second  risehour  riseminuter  sethour  setminute  \n",
       "0      9   29    23      55      26         6           13       18         13  \n",
       "1      9   29    23      50      23         6           13       18         13  \n",
       "2      9   29    23      45      26         6           13       18         13  \n",
       "3      9   29    23      40      21         6           13       18         13  \n",
       "4      9   29    23      35      24         6           13       18         13  "
      ]
     },
     "execution_count": 102,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# glimpse of the final data\n",
    "df.head()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 137,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "array([1.21, 1.21, 1.23, ..., 1.2 , 1.23, 1.2 ])"
      ]
     },
     "execution_count": 137,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "np.array(df['Radiation'])"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 136,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "(178,)"
      ]
     },
     "execution_count": 136,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "y.shape"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 153,
   "metadata": {},
   "outputs": [],
   "source": [
    "input_features = df.drop('Radiation', axis = 1)\n",
    "target = df['Radiation']"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Feature Selection using Correlation Matrix"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "$$\n",
    "r=\\frac{\\sum\\left(x_i-\\bar{x}\\right)\\left(y_i-\\bar{y}\\right)}{\\sqrt{\\sum\\left(x_i-\\bar{x}\\right)^2 \\sum\\left(y_i-\\bar{y}\\right)^2}}\n",
    "$$\n",
    "- $r=$ correlation coefficient\n",
    "- $x_i=$ values of the $\\mathrm{x}$-variable in a sample\n",
    "- $\\bar{x}=$ mean of the values of the $\\mathrm{x}$-variable\n",
    "- $y_i=$ values of the $y$-variable in a sample\n",
    "- $\\bar{y}=$ mean of the values of the $y$-variable"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 108,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Radiation</th>\n",
       "      <th>Temperature</th>\n",
       "      <th>Pressure</th>\n",
       "      <th>Humidity</th>\n",
       "      <th>WindDirection(Degrees)</th>\n",
       "      <th>Speed</th>\n",
       "      <th>Month</th>\n",
       "      <th>Day</th>\n",
       "      <th>Hour</th>\n",
       "      <th>Minute</th>\n",
       "      <th>Second</th>\n",
       "      <th>risehour</th>\n",
       "      <th>riseminuter</th>\n",
       "      <th>sethour</th>\n",
       "      <th>setminute</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>Radiation</th>\n",
       "      <td>1.000000</td>\n",
       "      <td>0.734955</td>\n",
       "      <td>0.119016</td>\n",
       "      <td>-0.226171</td>\n",
       "      <td>-0.230324</td>\n",
       "      <td>0.073627</td>\n",
       "      <td>-0.095450</td>\n",
       "      <td>0.039978</td>\n",
       "      <td>0.004398</td>\n",
       "      <td>-0.000730</td>\n",
       "      <td>-0.031270</td>\n",
       "      <td>NaN</td>\n",
       "      <td>-0.092850</td>\n",
       "      <td>0.048719</td>\n",
       "      <td>-0.039816</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Temperature</th>\n",
       "      <td>0.734955</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>0.311173</td>\n",
       "      <td>-0.285055</td>\n",
       "      <td>-0.259421</td>\n",
       "      <td>-0.031458</td>\n",
       "      <td>-0.354560</td>\n",
       "      <td>-0.123705</td>\n",
       "      <td>0.197464</td>\n",
       "      <td>-0.001934</td>\n",
       "      <td>-0.036147</td>\n",
       "      <td>NaN</td>\n",
       "      <td>-0.380968</td>\n",
       "      <td>0.300920</td>\n",
       "      <td>-0.242881</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Pressure</th>\n",
       "      <td>0.119016</td>\n",
       "      <td>0.311173</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>-0.223973</td>\n",
       "      <td>-0.229010</td>\n",
       "      <td>-0.083639</td>\n",
       "      <td>-0.341759</td>\n",
       "      <td>-0.024633</td>\n",
       "      <td>0.091069</td>\n",
       "      <td>0.001860</td>\n",
       "      <td>-0.031102</td>\n",
       "      <td>NaN</td>\n",
       "      <td>-0.380399</td>\n",
       "      <td>0.151939</td>\n",
       "      <td>-0.119599</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Humidity</th>\n",
       "      <td>-0.226171</td>\n",
       "      <td>-0.285055</td>\n",
       "      <td>-0.223973</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>-0.001833</td>\n",
       "      <td>-0.211624</td>\n",
       "      <td>-0.068854</td>\n",
       "      <td>0.014637</td>\n",
       "      <td>0.077899</td>\n",
       "      <td>0.000499</td>\n",
       "      <td>-0.027682</td>\n",
       "      <td>NaN</td>\n",
       "      <td>-0.023955</td>\n",
       "      <td>0.145143</td>\n",
       "      <td>-0.119526</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>WindDirection(Degrees)</th>\n",
       "      <td>-0.230324</td>\n",
       "      <td>-0.259421</td>\n",
       "      <td>-0.229010</td>\n",
       "      <td>-0.001833</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>0.073092</td>\n",
       "      <td>0.181485</td>\n",
       "      <td>-0.082354</td>\n",
       "      <td>-0.077969</td>\n",
       "      <td>-0.000602</td>\n",
       "      <td>-0.032568</td>\n",
       "      <td>NaN</td>\n",
       "      <td>0.176929</td>\n",
       "      <td>-0.078540</td>\n",
       "      <td>0.070030</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Speed</th>\n",
       "      <td>0.073627</td>\n",
       "      <td>-0.031458</td>\n",
       "      <td>-0.083639</td>\n",
       "      <td>-0.211624</td>\n",
       "      <td>0.073092</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>0.150822</td>\n",
       "      <td>0.117337</td>\n",
       "      <td>-0.057939</td>\n",
       "      <td>0.000192</td>\n",
       "      <td>-0.032934</td>\n",
       "      <td>NaN</td>\n",
       "      <td>0.167075</td>\n",
       "      <td>-0.159384</td>\n",
       "      <td>0.119926</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Month</th>\n",
       "      <td>-0.095450</td>\n",
       "      <td>-0.354560</td>\n",
       "      <td>-0.341759</td>\n",
       "      <td>-0.068854</td>\n",
       "      <td>0.181485</td>\n",
       "      <td>0.150822</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>0.038027</td>\n",
       "      <td>-0.005396</td>\n",
       "      <td>0.000168</td>\n",
       "      <td>0.220563</td>\n",
       "      <td>NaN</td>\n",
       "      <td>0.952472</td>\n",
       "      <td>-0.784783</td>\n",
       "      <td>0.541883</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Day</th>\n",
       "      <td>0.039978</td>\n",
       "      <td>-0.123705</td>\n",
       "      <td>-0.024633</td>\n",
       "      <td>0.014637</td>\n",
       "      <td>-0.082354</td>\n",
       "      <td>0.117337</td>\n",
       "      <td>0.038027</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>-0.008010</td>\n",
       "      <td>-0.000196</td>\n",
       "      <td>0.089078</td>\n",
       "      <td>NaN</td>\n",
       "      <td>0.274522</td>\n",
       "      <td>-0.263575</td>\n",
       "      <td>0.265662</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Hour</th>\n",
       "      <td>0.004398</td>\n",
       "      <td>0.197464</td>\n",
       "      <td>0.091069</td>\n",
       "      <td>0.077899</td>\n",
       "      <td>-0.077969</td>\n",
       "      <td>-0.057939</td>\n",
       "      <td>-0.005396</td>\n",
       "      <td>-0.008010</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>-0.004052</td>\n",
       "      <td>0.004199</td>\n",
       "      <td>NaN</td>\n",
       "      <td>-0.006772</td>\n",
       "      <td>0.008629</td>\n",
       "      <td>-0.007056</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Minute</th>\n",
       "      <td>-0.000730</td>\n",
       "      <td>-0.001934</td>\n",
       "      <td>0.001860</td>\n",
       "      <td>0.000499</td>\n",
       "      <td>-0.000602</td>\n",
       "      <td>0.000192</td>\n",
       "      <td>0.000168</td>\n",
       "      <td>-0.000196</td>\n",
       "      <td>-0.004052</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>0.002517</td>\n",
       "      <td>NaN</td>\n",
       "      <td>-0.000158</td>\n",
       "      <td>0.001052</td>\n",
       "      <td>-0.002215</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Second</th>\n",
       "      <td>-0.031270</td>\n",
       "      <td>-0.036147</td>\n",
       "      <td>-0.031102</td>\n",
       "      <td>-0.027682</td>\n",
       "      <td>-0.032568</td>\n",
       "      <td>-0.032934</td>\n",
       "      <td>0.220563</td>\n",
       "      <td>0.089078</td>\n",
       "      <td>0.004199</td>\n",
       "      <td>0.002517</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>NaN</td>\n",
       "      <td>0.258917</td>\n",
       "      <td>-0.037743</td>\n",
       "      <td>0.003571</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>risehour</th>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>riseminuter</th>\n",
       "      <td>-0.092850</td>\n",
       "      <td>-0.380968</td>\n",
       "      <td>-0.380399</td>\n",
       "      <td>-0.023955</td>\n",
       "      <td>0.176929</td>\n",
       "      <td>0.167075</td>\n",
       "      <td>0.952472</td>\n",
       "      <td>0.274522</td>\n",
       "      <td>-0.006772</td>\n",
       "      <td>-0.000158</td>\n",
       "      <td>0.258917</td>\n",
       "      <td>NaN</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>-0.742329</td>\n",
       "      <td>0.562851</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>sethour</th>\n",
       "      <td>0.048719</td>\n",
       "      <td>0.300920</td>\n",
       "      <td>0.151939</td>\n",
       "      <td>0.145143</td>\n",
       "      <td>-0.078540</td>\n",
       "      <td>-0.159384</td>\n",
       "      <td>-0.784783</td>\n",
       "      <td>-0.263575</td>\n",
       "      <td>0.008629</td>\n",
       "      <td>0.001052</td>\n",
       "      <td>-0.037743</td>\n",
       "      <td>NaN</td>\n",
       "      <td>-0.742329</td>\n",
       "      <td>1.000000</td>\n",
       "      <td>-0.873471</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>setminute</th>\n",
       "      <td>-0.039816</td>\n",
       "      <td>-0.242881</td>\n",
       "      <td>-0.119599</td>\n",
       "      <td>-0.119526</td>\n",
       "      <td>0.070030</td>\n",
       "      <td>0.119926</td>\n",
       "      <td>0.541883</td>\n",
       "      <td>0.265662</td>\n",
       "      <td>-0.007056</td>\n",
       "      <td>-0.002215</td>\n",
       "      <td>0.003571</td>\n",
       "      <td>NaN</td>\n",
       "      <td>0.562851</td>\n",
       "      <td>-0.873471</td>\n",
       "      <td>1.000000</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "                        Radiation  Temperature  Pressure  Humidity  \\\n",
       "Radiation                1.000000     0.734955  0.119016 -0.226171   \n",
       "Temperature              0.734955     1.000000  0.311173 -0.285055   \n",
       "Pressure                 0.119016     0.311173  1.000000 -0.223973   \n",
       "Humidity                -0.226171    -0.285055 -0.223973  1.000000   \n",
       "WindDirection(Degrees)  -0.230324    -0.259421 -0.229010 -0.001833   \n",
       "Speed                    0.073627    -0.031458 -0.083639 -0.211624   \n",
       "Month                   -0.095450    -0.354560 -0.341759 -0.068854   \n",
       "Day                      0.039978    -0.123705 -0.024633  0.014637   \n",
       "Hour                     0.004398     0.197464  0.091069  0.077899   \n",
       "Minute                  -0.000730    -0.001934  0.001860  0.000499   \n",
       "Second                  -0.031270    -0.036147 -0.031102 -0.027682   \n",
       "risehour                      NaN          NaN       NaN       NaN   \n",
       "riseminuter             -0.092850    -0.380968 -0.380399 -0.023955   \n",
       "sethour                  0.048719     0.300920  0.151939  0.145143   \n",
       "setminute               -0.039816    -0.242881 -0.119599 -0.119526   \n",
       "\n",
       "                        WindDirection(Degrees)     Speed     Month       Day  \\\n",
       "Radiation                            -0.230324  0.073627 -0.095450  0.039978   \n",
       "Temperature                          -0.259421 -0.031458 -0.354560 -0.123705   \n",
       "Pressure                             -0.229010 -0.083639 -0.341759 -0.024633   \n",
       "Humidity                             -0.001833 -0.211624 -0.068854  0.014637   \n",
       "WindDirection(Degrees)                1.000000  0.073092  0.181485 -0.082354   \n",
       "Speed                                 0.073092  1.000000  0.150822  0.117337   \n",
       "Month                                 0.181485  0.150822  1.000000  0.038027   \n",
       "Day                                  -0.082354  0.117337  0.038027  1.000000   \n",
       "Hour                                 -0.077969 -0.057939 -0.005396 -0.008010   \n",
       "Minute                               -0.000602  0.000192  0.000168 -0.000196   \n",
       "Second                               -0.032568 -0.032934  0.220563  0.089078   \n",
       "risehour                                   NaN       NaN       NaN       NaN   \n",
       "riseminuter                           0.176929  0.167075  0.952472  0.274522   \n",
       "sethour                              -0.078540 -0.159384 -0.784783 -0.263575   \n",
       "setminute                             0.070030  0.119926  0.541883  0.265662   \n",
       "\n",
       "                            Hour    Minute    Second  risehour  riseminuter  \\\n",
       "Radiation               0.004398 -0.000730 -0.031270       NaN    -0.092850   \n",
       "Temperature             0.197464 -0.001934 -0.036147       NaN    -0.380968   \n",
       "Pressure                0.091069  0.001860 -0.031102       NaN    -0.380399   \n",
       "Humidity                0.077899  0.000499 -0.027682       NaN    -0.023955   \n",
       "WindDirection(Degrees) -0.077969 -0.000602 -0.032568       NaN     0.176929   \n",
       "Speed                  -0.057939  0.000192 -0.032934       NaN     0.167075   \n",
       "Month                  -0.005396  0.000168  0.220563       NaN     0.952472   \n",
       "Day                    -0.008010 -0.000196  0.089078       NaN     0.274522   \n",
       "Hour                    1.000000 -0.004052  0.004199       NaN    -0.006772   \n",
       "Minute                 -0.004052  1.000000  0.002517       NaN    -0.000158   \n",
       "Second                  0.004199  0.002517  1.000000       NaN     0.258917   \n",
       "risehour                     NaN       NaN       NaN       NaN          NaN   \n",
       "riseminuter            -0.006772 -0.000158  0.258917       NaN     1.000000   \n",
       "sethour                 0.008629  0.001052 -0.037743       NaN    -0.742329   \n",
       "setminute              -0.007056 -0.002215  0.003571       NaN     0.562851   \n",
       "\n",
       "                         sethour  setminute  \n",
       "Radiation               0.048719  -0.039816  \n",
       "Temperature             0.300920  -0.242881  \n",
       "Pressure                0.151939  -0.119599  \n",
       "Humidity                0.145143  -0.119526  \n",
       "WindDirection(Degrees) -0.078540   0.070030  \n",
       "Speed                  -0.159384   0.119926  \n",
       "Month                  -0.784783   0.541883  \n",
       "Day                    -0.263575   0.265662  \n",
       "Hour                    0.008629  -0.007056  \n",
       "Minute                  0.001052  -0.002215  \n",
       "Second                 -0.037743   0.003571  \n",
       "risehour                     NaN        NaN  \n",
       "riseminuter            -0.742329   0.562851  \n",
       "sethour                 1.000000  -0.873471  \n",
       "setminute              -0.873471   1.000000  "
      ]
     },
     "execution_count": 108,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# extract the correlation between the data features\n",
    "corr_matrix = df.corr()\n",
    "corr_matrix"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 69,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "iVBORw0KGgoAAAANSUhEUgAABJ0AAATQCAYAAABZdjT1AAAAOXRFWHRTb2Z0d2FyZQBNYXRwbG90bGliIHZlcnNpb24zLjMuMywgaHR0cHM6Ly9tYXRwbG90bGliLm9yZy/Il7ecAAAACXBIWXMAAAsTAAALEwEAmpwYAAEAAElEQVR4nOzdd3wU1frH8c/Z9EBCCglJqKEpSG8i0kP3qtg7ihVFUaRIvVYU9WfDqygW1GvBem0gVRGkSO+I9JZGEkIC6dn5/bFLSCD0zW6I3/frxYvszJnZ58nMnp08e+assSwLERERERERERERV7J5OgAREREREREREal4VHQSERERERERERGXU9FJRERERERERERcTkUnERERERERERFxORWdRERERERERETE5VR0EhERERERERERl1PRSURERERERESkAjDGfGiMSTbGbDjJemOMmWSM2WaMWWeMaVVs3Z3GmK3Of3e6Ih4VnUREREREREREKoaPgD6nWN8XaOD8dz8wGcAYEwY8CVwKtAOeNMaEnm8wKjqJiIiIiIiIiFQAlmUtANJO0eRq4BPLYSkQYoyJBnoDcyzLSrMs6yAwh1MXr86Iik4iIiIiIiIiIv8M1YG9xR7vcy472fLz4n2+O5CKKT9lh+XpGNyl4M8fPR2CW+0cMsfTIbjNzwXnPRr0gmL3dABu1Mue6ekQ3GprQWVPh+A2NcnxdAhulWP38nQIUkby/mGf7fZJmubpENzmy+jbPB2C29Qx2Z4Owa1SCv08HYJbXZH0hfF0DGWpov5N6xtR7wEct8UdNcWyrCmeiud0VHQSEREREREREbkAOAtM51Nk2g/ULPa4hnPZfqDrccvnn8fzALq9TkRERERERETkn+JHYIDzW+zaA4csy0oAZgG9jDGhzgnEezmXnReNdBIRERERERERqQCMMV/gGLFU1RizD8c30vkAWJb1DjAD6AdsA7KAgc51acaYZ4Hlzl09Y1nWqSYkPyMqOomIiIiIiIhIxWIv9HQEHmFZ1i2nWW8Bg0+y7kPgQ1fGo9vrRERERERERETE5VR0EhERERERERERl1PRSUREREREREREXE5FJxERERERERERcTlNJC4iIiIiIiIiFYtl93QEgkY6iYiIiIiIiIhIGVDRSUREREREREREXE5FJxERERERERERcTnN6SQiIiIiIiIiFYtd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voq9wM6WUR+c8EUP5UXq3NWEx7Xisj8nUZidx+ZH3y5a13beSyyPGwnAlifep9Gkh/Dy9yV13hpS5zm+4Wr3m9/T5L2hRN/anZx9B9hw32un3a9/zQj8Y6qSvnjTsUCModGbg/EOCgQDhzfuZsvI993wG3A4MHc1VeNa0PnPNyjMzmX9o+8UreswbyKL40YBsOmJD2k66UG8/H05MG8NKfPWANBw3C1Uqh8DdjvZ+1LYOMIRu29EFTrMfh7voAAsu0Wd+/uysNPwEhOPS9kqq3656qUX0ejhK4/1y6Onkpd22M3ZFVNoZ9fY97no839jvGwcmDaP7L/3Un3EzRxZu5302ctJ/mIe9SY9SvNFb1GQfphtD7562t02eH8EPqFB2PML2TXmPQozyueUFMlzVxMR14Kuf75OYXYu6x49dpw7znuBP+JGA7Dhiak0nzQIm/M1fMD5GpbzYK94t1xeiEx5/3Ys8Yz8lB3/mBOj4M8fT9+oAtk5ZM7pG1UQPxecfMLriuif9Lbay555+kYVyNaCyp4OwW1qUvHnTyoux+7l6RCkjOT9w24o6JM0zdMhuM2X0cd/l0/FVcf8s4qQKYV+ng7Bra5I+qJcfaO5q+XtWFYh/6b1rdvugjpu/6x3QxERERERERERcQsVnURERERERERExOVUdBIREREREREREZfTROIiIiIiIiIiUqFY1j9pxtPySyOdRERERERERETE5VR0EhERERERERERl1PRSUREREREREREXE5zOomIiIiIiIhIxWLXnE7lgUY6iYiIiIiIiIiIy6noJCIiIiIiIiIiLqeik4iIiIiIiIiIuJzmdBIRERERERGRisXSnE7lgUY6iYiIiIiIiIiIy6noJCIiIiIiIiIiLqeik4iIiIiIiIiIuJzmdBIRERERERGRisVe6OkIBI10EhERERERERGRMqCik4iIiIiIiIiIuJyKTiIiIiIiIiIi4nIqOomIiIiIiIiIiMtpInERERERERERqVgsu6cjEDTSSUREREREREREyoCKTiIiIiIiIiIi4nIqOomIiIiIiIiIiMtpTicRERERERERqVjsmtOpPNBIJxERERERERERcTkVnURERERERERExOVUdBIREREREREREZfTnE4iIiIiIiIiUrFYmtOpPNBIJxERERERERERcTkVnURERERERERExOVUdBIREREREREREZfTnE5SqoI/f/R0CG7jfelVng7BraKa/M/TIbhNxtoqng7BrXwt4+kQ3CY5L8DTIbjVwoACT4fgNu3y/1nHtmv1BE+H4Dbb9oZ7OgS30kwiFdevfnmeDsFtnm+a6ukQ3OrXdTU9HYJbXeHpAMqaXT1xeaCRTiIiIiIiIiIi4nIqOomIiIiIiIiIiMup6CQiIiIiIiIiIi6nopOIiIiIiIiIiLicJhIXERERERERkQrFsgo9HYKgkU4iIiIiIiIiIlIGVHQSERERERERERGXU9FJRERERERERERcTnM6iYiIiIiIiEjFYtk9HYGgkU4iIiIiIiIiIlIGVHQSERERERERERGXU9FJRERERERERERcTnM6iYiIiIiIiEjFYtecTuWBRjqJiIiIiIiIiIjLqegkIiIiIiIiIiIup6KTiIiIiIiIiIi4nOZ0EhEREREREZGKxdKcTuWBRjqJiIiIiIiIiIjLqegkIiIiIiIiIiIup6KTiIiIiIiIiIi4nIpOIiIiIiIiIiLicppIXEREREREREQqFnuhpyMQNNJJRERERERERETKgIpOIiIiIiIiIiLicio6iYiIiIiIiIiIy2lOJxERERERERGpWCy7pyMQNNJJRERERERERETKgIpOIiIiIiIiIiLicio6iYiIiIiIiIiIy2lOJxERERERERGpWOya06k80EgnERERERERERFxuX/8SCdjTDgwz/kwCigEDjgft7MsK88jgZXCGNMVyLMsa7GHQzlrizbv4aXv/8But7imfSPujmtVYv3L3y9i+bb9AOTkF5CWmc0fz99DfFomj0+did2yKCi0c0unptzQ4RJPpOAy455/lQWLlhEWGsL3n77j6XDOm0/LdgTe9wjYbOTOmU7Ot5+XWO/X5yr8+l4D9kKsnGyOvP1/2PfuxgQFU/mJZ/CufxG5v84ka8obHsrg7F3x5AAadmtBfnYe3w5/h4SNu0qs9/H35ea3HyWsdjXshXa2zFvF7BenARBSvSrXvHQ/lcKCyT50mK8fe5uMxDQPZHFmej81gAbdmpOfnccPw98lccOuEuu9/X25YfIQQmtVw263s3XuKua9+GXR+sZXXEqXoddhWRZJm/fwvyFvuTmD02s04U6qxrXEnp3L+iGTyVi/64Q2wc1iaTrpQWz+vqTMW83msR8DUP+JG6nWpzWW3SIv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IuJyKjqJiIiIiIiIiIjLqegkIiIiIiIiIiIupzmdRERERERERKRisWtOp/JAI51ERERERERERMTlVHQSERERERERERGXU9FJRERERERERERcTnM6iYiIiIiIiEjFYmlOp/JAI51ERERERERERMTlVHQSERERERERERGXU9FJRERERERERERcTnM6iYiIiIiIiEjFYtecTuWBRjqJiIiIiIiIiFQQxpg+xpgtxphtxphRpax/zRizxvnvb2NMerF1hcXW/Xi+sWikk4iIiIiIiIhIBWCM8QLeAnoC+4DlxpgfLcvadLSNZVlDi7V/BGhZbBfZlmW1cFU8GukkIiIiIiIiIlIxtAO2WZa1w7KsPGAacPUp2t8CfFFWwWikk5Rq55A5ng7BbaKa/M/TIbhV0AdTPR2C2wS3/renQ5Aykmnz8nQIbuVFoadDcJvIggJPh+BWaQcqeToEKSP6ZFcqgoPb/T0dglvVzPd0BCIuUR3YW+zxPuDS0hoaY2oDscCvxRb7G2NWAAXARMuyvj+fYFR0EhEREREREZGKxaqYE4kbY+4H7i+2aIplWVPOcXc3A99YllX8E87almXtN8bUBX41xqy3LGv7ucaropOIiIiIiIiIyAXAWWA6VZFpP1Cz2OMazmWluRkYfNz+9zv/32GMmY9jvqdzLjpp5K+IiIiIiIiISMWwHGhgjIk1xvjiKCyd8C10xpiLgVBgSbFlocYYP+fPVYHLgU3Hb3s2NNJJRERERERERKQCsCyrwBjzMDAL8AI+tCxrozHmGWCFZVlHC1A3A9Msy7KKbd4IeNcYY8cxSGli8W+9OxcqOomIiIiIiIhIxWKvmHM6nQnLsmYAM45b9u/jHj9VynaLgaaujEW314mIiIiIiIiIiMup6CQiIiIiIiIiIi6nopOIiIiIiIiIiLic5nQSERERERERkYrlHzynU3mikU4iIiIiIiIiIuJyKjqJiIiIiIiIiIjLqegkIiIiIiIiIiIupzmdRERERERERKRisTSnU3mgkU4iIiIiIiIiIuJyKjqJiIiIiIiIiIjLqegkIiIiIiIiIiIup6KTiIiIiIiIiIi4nCYSFxEREREREZGKxa6JxMsDjXQSERERERERERGXU9FJRERERERERERcTkUnERERERERERFxOc3pJCIiIiIiIiIVi6U5ncoDjXQSERERERERERGX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TnM6iYiIiIiIiEjFoqmEygWNdBIREREREREREZdT0UlERERERERERFxORScREREREREREXE5zekkIiIiIiIiIhWL3e7pCASNdBIRERERERERkTKgopOIiIiIiIiIiLicik4iIiIiIiIiIuJyKjqJiIiIiIiIiIjLaSJxEREREREREalYNJF4uaCRTiIiIiIiIiIi4nIqOomIiIiIiIiIiMup6CQiIiIiIiIiIi6nOZ1EREREREREpGKxNKdTeaCRTiIiIiIiIiIi4nIqOomIiIiIiIiIiMup6CQiIiIiIiIiIi6nOZ1EREREREREpGKxa06n8kAjnURERERERERExOVUdBIREREREREREZdT0UlERERERERERFxOczqJiIiIiIiISMViWZ6OQNBIJxERERERERERKQMqOomIiIiIiIiIiMup6CQiIiIiIiIiIi6nopOIiIiIiIiIiLicJhIXERERERERkYrFbvd0BIJGOomIiIiIiIiISBlQ0UlERERERERERFxORScREREREREREXE5zekkpfq5INTTIbhNxtoqng7BrYJb/9vTIbjN0JXPeDoEt1p4yShPh+A2q/18PR2CW3XJ+ed8RhRo8j0dglvZ7cbTIYjIWbou28vTIbiNPUB9lFzANKdTufDPuYoVERERERERERG3UdFJRERERERERERcTkUnERERERERERFxOc3pJCIiIiIiIiIVi6U5ncoDjXQSERERERERERGXU9FJRERERERERERcTkUnERERERERERFxOc3pJCIiIiIiIiIVimW3PB2CoJFOIiIiIiIiIiJSBlR0EhERERERERERl1PRSUREREREREREXE5FJxERERERERERcTlNJC4iIiIiIiIiFYvd7ukIBI10EhERERERERGRMqCik4iIiIiIiIiIuJyKTiIiIiIiIiIi4nKa00lEREREREREKhZLczqVBxrpJCIiIiIiIiIiLqeik4iIiIiIiIiIuJyKTiIiIiIiIiIi4nKa00lEREREREREKha75ekIBI10EhERERERERGRMqCik4iIiIiIiIiIuJyKTiIiIiIiIiIi4nKa00lEREREREREKha73dMRCBrpJCIiIiIiIiIiZUBFJxERERERERERcTkVnURERERERERExOVUdBIREREREREREZfTROIiIiIiIiIiUrFoIvFyQSOdRERE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87o0nfQgXv6+HJi3hs1jHJ8xdlr6OjZfH/Kdo9vSV25l08gP8Ar0o+kbD1KpYXWMMeybNp9db/98zjH2SZp2nlleOGZXu9ll+7r4hYFUdR7bjcWObft5E1la7Ng2mfQgNn9fUuat4S/nse3oPLZ5zmN7aOVWNo/8oMT+6w2/noIjOeyefG7Htk5Y+jlmdnKVOrclYvQDYLOR8d1s0t6dRvgjd5CzYStHfluKX5OGxLw5Hq/gIKy8PApSDrL7ygfwb9GIak8PAbsFNsPBT74n41vXFtOWHap6+kbnoO3zdxLT1XGtvGToFNKc1xf95kxgRs+xALQcdzN1+ncgMCqErMR0tn8xn3WvfEfcl6MIubhmUaEqa38q8+9yzbXy7fGfevjrZstW1qv3VchiR+Dj711Qx01Fp38Q57fnXW1Z1h2na1sWRafyqqyKTuVVWRSdyit3F5087XyKTheasio6lVeuKjpdCFxVdLpQuKLodKE4n6LThciVRacLgYpOFVNZFJ3Ks7IqOpVXKjpdmC60opPmdPqHMMa8CfQF+nk6FhERERERERGp+FR0+oewLOsRT8cgIiIiIiIi4hb2CjnQ6YLzzxr3KyIiIiIiIiIibqGik4iIiIiIiIiIuJyKTiIiIiIiIiIi4nIqOomIiIiIiIiIiMtpInERERERERERqVgsu6cjEDTSSUREREREREREyoCKTiIiIiIiIiIi4nIqOomIiIiIiIiIiMtpTicRERERERERqVjslqcjEDTSSUREREREREREyoCKTiIiIiIiIiIi4nIqOomIiIiIiIiIiMtpTicRERERERERqVAsu93TIQga6SQiIiIiIiIiImVARScREREREREREXE5FZ1ERERERERERMTlNKeTiIiIiIiIiFQsdsvTEQga6SQiIiIiIiIiImVARScREREREREREXE5FZ1ERERERERERMTlVHQSERERERERERGX00TiIiIiIiIiIlKxWHZPRyBopJOIiIiIiIiIiJQBFZ1ERERERERERMTlVHQSERERERERERGX05xOIiIiIiIiIlKx2C1PRyBopJOIiIiIiIiISIVhjOljjNlijNlmjBlVyvq7jDEHjDFrnP/uLbbuTmPMVue/O883Fo10EhERERERERGpAIwxXsBbQE9gH7DcGPOjZVmbjmv6pWVZDx+3bRjwJNAGsICVzm0Pnms8GukkIiIiIiIiIlIxtAO2WZa1w7KsPGAacPUZbtsbmGNZVpqz0DQH6HM+wWikk5TK7ukA3MjXMp4OQcrIwktOGElaoXXaONHTIbhNSMuhng7BrcKisjwdgtusSIr0dAhulVzg6+kQ3CaMfE+H4Fb/pGupf5oGUameDsFtVib/s/rki70OezoEcSX7P7Ynrg7sLfZ4H3BpKe2uM8Z0Bv4GhlqWtfck21Y/n2A00klERERERERE5AJgjLnfGLOi2L/7z2E3PwF1LMtqhmM008eujfIYjXQSEREREREREbkAWJY1BZhyiib7gZrFHtdwLiu+j+JDNt8HXiq2bdfjtp1/jqECGukkIiIiIiIiIlJRLAcaGGNijTG+wM3Aj8UbGGOiiz28Ctjs/HkW0MsYE2qMCQV6OZedM410EhEREREREZGKxW55OgKPsCyrwBjzMI5ikRfwoWVZG40xzwArLMv6ERhijLkKKADSgLuc26YZY57FUbgCeMayrLTziUdFJxERERERERGRCsKyrBnAjOOW/bvYz6OB0SfZ9kPgQ1fFotvrRERERERERETE5VR0EhERERERERERl1PRSUREREREREREXE5zOomIiIiIiIhIxWLZPR2BoJFOIiIiIiIiIiJSBlR0EhERERERERERl1PRSUREREREREREXE5zOomIiIiIiIhIxWK3PB2BoJFOIiIiIiIiIiJSBlR0EhERERERERERl1PRSUREREREREREXE5zOomIiIiIiIhIhWLZ7Z4OQdBIJxERERERERERKQMqOomIiIiIiIiIiMup6CQiIiIiIiIiIi6nOZ1EREREREREpGKxW56OQNBIJxERERERERERKQMqOomIiIiIiIiIiMup6CQiIiIiIiIiIi6nOZ1EREREREREpGLRnE7lgkY6iYiIiIiIiIiIy6noJCIiIiIiIiIiLqeik4iIiIiIiIiIuJyKTiIiIiIiIiIi4nKaSFxEREREREREKhbL7ukIBI10EhERERERERGRMnDakU7GmNeA3ZZlve58PAvYa1nWvc7HrwCHgDzLsiae6RMbYz4CfrYs6xtjzHwgGsgFfIG5wDjLstKdbRdbltXhzNM66XPeBcy2LCve+fh94FXLsjad4/4eA9Isy/rEmU8XIAMIAJYCYyzL2ne+cbuKMebo77a7ZVkF7n7+Hk/dQb1uLcjP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k4iIiIiIiIiIeJ0mEhcRERERERGRmsWlicSrA410EhERERERERERr1PRSUREREREREREvE5FJxERERERERER8TrN6SQiIiIiIiIiNYvmdKoWNNJJRERERERERES8TkUnERERERERERHxOhWdRERERERERETE6zSnk4iIiIiIiIjUKJbmdKoWVHSSSg105fk7BJ9JLwn1dwg+lWez+zsEn1keHOTvEHwqsuPd/g7BZ9otf9HfIfjU1p7D/R2Cz7QNy/V3CD61cV8df4cgIsfIchl/h+AzWfZ/Tq4AbYNL/R2CSI2j2+tERERERERERMTrVHQSERERERERERGv0+11IiIiIiIiIlKzaE6nakEjnURERERERERExOtUdBIREREREREREa9T0UlERERERERERLxORScREREREREREfE6TSQuIiIiIiIiIjWLy98BCGikk4iIiIiIiIiIVAEVnURERERERERExOtUdBIREREREREREa/TnE4iIiIiIiIiUqNYLsvfIQga6SQiIiIiIiIiIlVARScREREREREREfE6FZ1ERERERERERMTrNKeTiIiIiIiIiNQsmtOpWtBIJxERERERERER8ToVnURERERERERExOtUdBIREREREREREa/TnE4iIiIiIiIiUrO4/B2AgEY6iYiIiIiIiIhIFVDRSUREREREREREvE5FJxERERERERER8ToVnURERERERERExOs0kbiIiIiIiIiI1CiWy/J3CIJGOomIiIiIiIiISBVQ0UlERERERERERLxORScREREREREREfE6zekkIiIiIiIiIjWLy98BCGikk4iIiIiIiIiIVAEVnURERERERERExOt0e101YYyZAFwFOHEPBBxmWdYfVfS75gD3Wpa1pCr2/7eIPh1p9MitYLeR9ekM0l7/qmIcQQE0e+luQtu1wJmTx9bhz1GyK52oob2pf/vQsnahpzbjr3PuoXDt1rJlzd+bQHCT+qzrP6oqUzhmpz55PXWTOuIqLGb1qDfYu3rbQW0i2ifQ7pU7sIUEkTlrOesmvA9Ay/svo/7gzlgui5LMvawe9QbFaTnUatmAdi/fTkS7BDY8/Tnb3vjBx1lVLvHx64hP6oCjsITFo98it5JcI9s3o9tLt2MPCSRl1kpWTPwAgDqnNaXzMzdhDw7E5XSybNwUclZsKdsuqkNz+v3wCL/f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03Ronu7vEKSKNPrpOX+H4DMTTrvU3yH41I6JPfwdgog3NAR2lnu9CzjjMO1vBn4u9zrEGLMEcACTLMv69kSCUdFJREREREREROQkYIy5Dbit3KLJlmVNPs59XQN0AXqXW9zUsqzdxpjmwGxjzGrLsjYfb7wqOomIiIiIiIiInAQ8BabDFZl2A43LvW7kWVaBMaY/MAHobVlWcbn97/b8f4sxZg7QETjuotM/694TEREREREREZGaazHQyhiTYIwJAq4AKjyFzhjTEXgL+JdlWenllkcZY4I9P9cFzqbcXFDHQyOdRERERERERKRGsVz+jsA/LMtyGGPuBKYBduA9y7L+NMY8BiyxLOt74DmgNvClMQZgh2VZ/wJOBd4yxrhwD1KadMBT746Zik4iIiIiIiIiIjWEZVk/AT8dsOyhcj/3P8R2vwHtvBmLbq8TERERERERERGvU9FJRERERERERES8TrfXiYiIiIiIiEiNYrmMv0MQNNJJRERERERERESqgIpOIiIiIiIiIiLidSo6iYiIiIiIiIiI12lOJxERERERERGpUSyXvyMQ0EgnERERERERERGpAio6iYiIiIiIiIiI16noJCIiIiIiIiIiXqc5nURERERERESkRrEs4+8QBI10EhERERERERGRKqCik4iIiIiIiIiIeJ2KTiIiIiIiIiIi4nUqOomIiIiIiIiIiNdpInERERERERERqVEsl78jENBIJxERERERERERqQIqOomIiIiIiIiIiNep6CQiIiIiIiIiIl6nOZ1EREREREREpEaxXMbfIQga6SQiIiIiIiIiIlVARScREREREREREfE6FZ1ERERERERERMTrNKeTiIiIiIiIiNQoluXvCAQ00klERERERERERKqAik4iIiIiIiIiIuJ1ur2uGjDGWMDHlmVd43kdAKQAf1iWdd5x7C8SuMqyrNc9r/sA9x7Pvk5Ux8evIz6pA87CEhaNfouc1dsOahPVvhndXrode0ggKbNWsnziBwBEtm1C52duIqBWCPt2ZvD7iNdx5BcS1qgu58x7jrzNKQBkLdvE0vvf82VaR+WSh2/gtL4dKSks5sN732DXn1sPajP8/QeIiI3CbrexefFffD7xXSyXxZDRl9D9iiTys/cC8P2zn7J2zgofZ3BsLnr4etr27UhpYTEf3/sGu/7cdlCb298fR0RsFDa7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3kDpug0ULfiNiDuHYcJCiH7S/RQ+Z1o62WMfJDSpD8GJ7bFFRBA2xP1ezn3iGUo3bvZnRuIRYLczfuRN3D7uSZwuFxcO7kvLZo157T+fc1rrFvTt3oXFK9fy8rufYDB0bn8qE0a6R/ps3bGbR1+cjM1mw+VycfMVQ6tl0am8GdPm0H9gbxavnElhQSGjhj9Qtu6XBd9VeGrdBReewxWX3Fph+4fGP82Lrz7B7SNuxLIs7rxjnM9iP2aWi5KZHxF8yT1gs+FYvQArK5nAs4fiSt2Gc/MKAjv1d08u7nJhFeVT8vO7/o5axKvMyfq0LKlan8df/Y85MeYHO/wdgk/ZOfmq48erd9E/azBnp7jDP3mpJikptvs7BJ9KmP+6v0PwmcXt7jtyoxqk0PXP+f7PVMvnk1adNs2P7pbkmqLhwqp/qmF1UbLTfw8R8LUGp13q7xB8asfEHv4OwafC7nuvRv9hsC1xQI384Gm2YsZJddz+WX+RiYiIiIiIiIiIT/xzvl4TERERERERkX8E3dRVPWikk4iIiIiIiIiIeJ2KTiIiIiIiIiIi4nUqOomIiIiIiIiIiNdpTicRERERERERqVEs10n1kLcaSyOdRERERERERETE61R0EhERERERERERr1PRSUREREREREREvE5zOomIiIiIiIhIjWJZmtOpOtBIJxERERERERER8ToVnURERERERERExOtUdBIREREREREREa9T0UlERERERERERLxOE4mLiIiIiIiISI1iufwdgYBGOomIiIiIiIiISBVQ0UlERERERERERLxORScREREREREREfE6zekkIiIiIiIiIjWKyzL+DkHQSCcREREREREREakCKjqJiIiIiIiIiIjXqegkIiIiIiIiIiJepzmdRERERERERKRGsTSnU7WgkU4iIiIiIiIiIuJ1KjqJiIiIiIiIiIjXqegkIiIiIiIiIiJepzmdRERERERERKRGsVya06k60EgnERERERERERHxOhWdRERERERERETE61R0EhERERERERERr1PRSUREREREREREvE4TiYuIiIiIiIhIjWJZ/o5AQEUnOYTGFPk7BJ/pVhrq7xB8Ktbh8HcIPhNmSv0dgk8tSYv1dwg+0zYs198h+NTidvf5OwSf6br6OX+H4FPzTnvA3yFIFclIru3vEHyqob8D8KE9N43ydwg+Myqqq79D8Kn8qZv9HYJPhf1zLi/Ej3R7nYiIiIiIiIiIeJ2KTiIiIiIiIiIi4nW6vU5EREREREREahTLZfwdgqCRTiIiIiIiIiIiUgVUdBIREREREREREa9T0UlERERERERERLxOczqJiIiIiIiISI3isjSnU3WgkU4iIiIiIiIiIuJ1KjqJiIiIiIiIiIjXqegkIiIiIiIiIiJepzmdRERERERERKRGsTSnU7WgkU4iIiIiIiIiIuJ1KjqJiIiIiIiIiIjXqegkIiIiIiIiIiJep6KTiIiIiIiIiIh4nSYSFxEREREREZEaxbL8HYGARjqJiIiIiIiIiEgVUNFJRERERERERES8TkUnERERERERERHxOs3pJCIiIiIiIiI1issy/g5B0EgnERERERERERGpAio6iYiIiIiIiIiI16noJCIiIiIiIiIiXqc5nURERERERESkRrE0p1O1oJFOIiIiIiIiIiLidSo6iYiIiIiIiIiI16noJCIiIiIiIiIiXqc5nURERERERESkRrEsf0cgoJFOIiIiIiIiIiJSBVR0EhERERERERERr1PRSUREREREREREvE5zOp1kjDFOYDUQCDiAD4AXLcty+TWwI4jsm0jCYzeB3Ub6J7PY/do3FdaboABavTKKWu2b48jJY8OwFyjelVG2PqhhXTrOfYmd//cFyW9+7+vwj0vXx66lYb9EnIXF/Hr3ZLLXbDuoTeL9l9Likh4E1anFp61vKVt+6m3n0OrKPlgOJ0XZefx2z2T27c7yYfRHduqT11M3qSOuwmJWj3qDvau3HdQmon0C7V65A1tIEJmzlrNuwvsAtHnoauoN7IRV6qBgWxqr73oTx94CQhvXo8f859m3ORmA3KUbWTv2XV+mVamovom0ePxGjN1G6sez2PnatxXWm6AA2rw6kvD2zSnNyWPdsBcp3plBQFRt2r4zhvDElqR+PofN4/fn0v7rRwiKjcJVVALA6isepzRzry/TOqQOj19HfFIHHIUlLBn9FrmVHNvI9s3o+tLt2EMCSZm1kpUTPwCgzmlN6fTMTdiDA3E5nSwfN4WcFVtofFF32ow4H2MMjvxClo2bwp61O3yc2eHV6tWZuIm3Yew2cj6fTtZbX1ZYH9b1NOo/eBshpySw665nyJv6KwDBpzYn/rHh2GqHgctF5uufs/fH+f5I4bCOtx+undiSFs/d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qjbt3h5NSMO6FO3KZPWtL+LI3UdQ/SjavnIHwfWjwBi2v/ItqV8d3+O6DSd2adH66ZuJ6dcBZ2EJ6+56vSy/rrOeZXHS2LL8Tn1lOPaQIE9+75Xld/rbd3vyy2CNJ79D7bf+xT059eU72Ld+fyF13ah/k//n9qOONzrkxKcGCO/bmYYP3YKx28j+YiZpr31J3D1XUbBqE3tnLsIEB9L0xXsIPa05jtw8tt/5HCU707CFhdDk/+4iuFVjjIGsL2eR8dY3ALT88mkCosKxSp3sfuJd8n9ddcJxAiRu9+71WXWWMaC31/cZ1O0Mat0xEmOzUTTtJwo++Yiw62/CseEvShb+Rp1nnicgoTmubPdoN2d6Onsfcn8m2+rHEfnSa2RfdanXbwN9Y0OjIzc6Duc8fgMterentLCE7+99i5TV7muN2356islDxhMeF83df7xKxqbdOIvdnyuLP5jO8s/mADBw4tW06N0Bl9PFgte+5c///e6VuG5v6f0vT4K6nUHtEe5jW/iz+9jWuuEmSte7j23ks88T0Lw5ziz3sXWlp7Nn4v5jG/XKa2Rd4f1jCxA7a26NHgq0uOGFNbLY0XX3NyfVcVPR6R/CGBOGu1jVybKsPUdq7+2iU3VWVUWn6spbRaeTgbeKTicLbxWdTgZVUXSqzrxRdDpZ+Lro5G/eLDpVdydadDrZeKPodDJR0almqqqiU3VVFUWn6kxFp5PTyVZ00pxO/wDGmP64n2D34tEUnEREREREREROZi7N6VQtqOj0D2BZ1kygqb/jEBEREREREZF/Dk0kLiIiIiIiIiIiXqeik4iIiIiIiIiIeJ1urxMRERERERGRGqVGziJ+EtJIJxERERERERER8ToVnURERERERERExOtUdBIREREREREREa9T0UlERERERERERLxOE4mLiIiIiIiISI3isoy/QxA00klERERERERERKqAik4iIiIiIiIiIuJ1KjqJiIiIiIiIiIjXaU4nEREREREREalRLM3pVC1opJOIiIiIiIiIiHidik4iIiIiIiIiIuJ1KjqJiIiIiIiIiIjXaU4nEREREREREalRXP4OQACNdBIRERERERERkSqgopOIiIiIiIiIiHidik4iIiIiIiIiIuJ1mtNJRERERERERGoUC+PvEASNdBIRERERERERkSqgopOIiIiIiIiIiHidik4iIiIiIiIiIuJ1KjqJiIiIiIiIiIjXaSJxEREREREREalRXJa/IxDQSCcREREREREREakCKjqJiIiIiIiIiIjXqegkIiIiIiIiIlJDGGMGG2PWG2M2GWPGVbI+2BjzuWf9H8aYZuXWPeBZvt4YM+hEY9GcTiIiIiIiIiJSo7gw/g7BL4wxduDfwABgF7DYGPO9ZVlryzW7GcixLKulMeYK4BngcmNMW+AK4DSgATDTGNPasizn8cajopNUqshl93cIPtOnYYq/Q/Cp7Ixa/g7BZ1yuf9YHTbojyN8h+MzGfXX8HYJP1T7+z/mTzrzTHvB3CD7V68+n/R2Cz/zTjm12UYi/Q5AqsmpVnL9D8JlelPg7BJ9aveafc2wBkvwdgFSVbsAmy7K2ABhjPgMuAMoXnS4AHvH8/F/gNWOM8Sz/zLKsYmCrMWaTZ38LjzcY3V4nIiIiIiIiInISMMbcZoxZUu6/2w5o0hDYWe71Ls+ySttYluUA9gAxR7ntMdFIJxERERERERGRk4BlWZOByf6O42ip6CQiIiIiIiIiNYr1D53TCdgNNC73upFnWWVtdhljAoA6QNZRbntMdHudiIiIiIiIiEjNsBhoZYxJMMYE4Z4Y/PsD2nwPXO/5+RJgtmVZlmf5FZ6n2yUArYBFJxKMRjqJiIiIiIiIiNQAlmU5jDF3AtMAO/CeZVl/GmMeA5ZYlvU98C7woWei8GzchSk87b7APem4AxhxIk+uAxWdRERERERERERqDMuyfgJ+OmDZQ+V+LgIuPcS2TwJPeisWFZ1EREREREREpEZx+TsAATSnk4iIiIiIiIiIVAEVnURERERERERExOtUdBIREREREREREa9T0UlERERERERERLxOE4mLiIiIiIiISI1iYfwdgqCRTiIiIiIiIiIiUgVUdBIREREREREREa9T0UlERERERERERLxOczqJiIiIiIiISI3i8ncAAmikk4iIiIiIiIiIVAEVnURERERERERExOtUdBIREREREREREa/TnE4iIiIiIiIiUqNoTqfqQSOdRERERERERETE61R0EhERERERERERr1PRSUREREREREREvE5zOomIiIiIiIhIjWJh/B2CoJFOIiIiIiIiIiJSBVR0EhERERERERERr1PRSUREREREREREvE5FJxERERERERER8TpNJC4iIiIiIiIiNYpL84hXCxrp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P1DRSUREREREREREfE5FJxERERERERER8TnN6SQiIiIiIiIiNYrLBDsCAfV0EhERERERERERP1BPJznubdgWH+wQRHwijopghyAiR+n3k8cGO4SA6b3y6WCHEFAzTx4T7BDET8xx9J1YLo6vriLH07EVCRT1dBIREREREREREZ9TTycRERERERERqVFcwQ5AAPV0EhERERERERERP1DRSUREREREREREfE5FJxERERERERER8TnN6SQiIiIiIiIiNYq+i7B6UE8nERERERERERHxORWdRERERERERETE51R0EhERERERERERn1PRSUREREREREREfE4TiYuIiIiIiIhIjeIywY5AQD2dRERERERERETED1R0EhERERERERERn1PRSUREREREREREfE5zOomIiIiIiIhIjeIKdgACqKeTiIiIiIiIiIj4gYpOIiIiIiIiIiLicyo6iYiIiIiIiIiIz2lOJxERERERERGpUTSnU/Wgnk4iIiIiIiIiIuJzKjqJiIiIiIiIiIjPqegkIiIiIiIiIiI+pzmdRERERERERKRGsUywIxBQTycREREREREREfEDFZ1ERERERERERMTnVHQSERERERERERGfU9FJRERERERERER8ThOJi4iIiIiIiEiN4gp2AAKop5OIiIiIiIiIiPiBik4iIiIiIiIiIuJzKjqJiIiIiIiIiIjPaU6nIDHGWMCHlmVd4XkcAqQD8yzL+pcx5hygtWVZ4/7m/kcBEy3LKvZVzIfS4slrie/fEVdJGatGvkrRn5v2axPTrgknvTQcW0QYuVOWsP7+dwEIqRNNm4l3ENEwkdJt2ay48QUcu3YfdL91epxMi8eu9u43qnkqK4e9SM5PCzjxhWHEtG+KMYbijemsHvl/OIvLjrnco5qnctKLtxLTtgkbn/6Eba99791XgxvPIvWK/oAh7cMpbJ84ya/57RHXtz0tnrgWY7eR/uEUtrz8bZX1JiyE1q/cRky7plTsLGTlTRMo3ZYNQKORQ0m5rB+W08X6+98lb/oyAE6ccAsJAzpRnrOL+aff7d1XrdaNaPXsjdijIyjdls3KW17CWVQSkDz38Ee+B9vnSS/eSp3urXEUuF+uq0f+H0UrtwQw28C+hgHC68dz0vPDCE+NBwuWXf6099/PHwJ5POtfN5CGN51NVJNkZp50PRV5he5/p9rRnDTh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OuL6tcdVUs6q21/z5nrKlGeY3/9eb66tX7oVW0SoJ9d3vbm2fXOUJ9cc/qyU6x4nvXgLuZMXk+UpqHX+9hGimtfHHh1Bxc5CVt/xhvfbY49E75VP/+1c/457Hh7HgiXLyc8vID6uDrdefyXnDxkYsOefefKYv7Wdr1/DtU9pRefvH6do1RYsl/ue6a+nPiZ3yhL3fto0BsuiZFs2a++eeNjCxcH0y/zsb213LJpa76Kjah/o99hO3z5a5bW65o7XyZu+jJDYKFq/OpKIBgk4d5ey9p43KVrlv2/MDeS9Y8JZXWn51HWExcfiKNhN4YrNLDvAt9z5kz+O84nP30zi2adSut09X5TlcLJw4NhDvq6PRr/Mz/w3qWY18HLDK2pksWPEtg+OqeOmopMc0NEWnUREROTo/ZOi07Em0EWnYPu7RadjlYpOIseeml50evGEmll0un3rsVV00vA6ERERERERERHxORWdRERERERERETE51R0EhERERERERERn1PRSUREREREREREfC4k2AGIiIiIiIiIiPiSK9gBCKCeTiIiIiIiIiIi4gcqOomIiIiIiIiIiM+p6CQiIiIiIiIiIj6nOZ1EREREREREpEbRnE7Vg3o6iYiIiIiIiIiIz6noJCIiIiIiIiIiPqeik4iIiIiIiIiI+JzmdBIRERERERGRGsUKdgACqKeTiIiIiIiIiIj4gYpOIiIiIiIiIiLicyo6iYiIiIiIiIiIz2lOJxERERERERGpUVwm2BEIqKeTiIiIiIiIiIj4gYpOIiIiIiIiIiLicyo6iYiIiIiIiIiIz6noJCIiIiIiIiIiPqeJxEVERERERESkRnEFOwAB1NNJRERERERERET8QEUnERERERERERHxORWdRERERERERETE5zSnk4iIiIiIiIjUKFawAxBAPZ1ERERERERERMQPVHQSERERERERERGfU9FJRERERERERER8TkUnEREREREREalRXFg18uefMMbEGWMmG2PWe/5f9wBtOhhj5hhjVhpjlhtjLq607j1jzCZjzFLPT4fDPaeKTiIiIiIiIiIiNd8YYIplWS2AKZ7H+yoGrrIs62RgEDDBGFOn0vp7LMvq4PlZergn1LfXyQGVH0f1SFewAwiw4+fIHn+Ot3P5eNK+aVawQwiY7LRawQ4hoPJKI4IdQsDMPPlA97U1V6+V44IdgvjJya0zgx1CwCxalRLsEAIqQndTUvOdC/Tx/P4+MB24t3IDy7LWVfo9zRiTBSQC+X/nCfX3p4iIiIiIiIjIMcAYc5MxZmGln5uOYvN6lmWle37PAOod5rlOAcKAjZUWP+kZdveCMSb8cE+onk4iIiIiIiIiUqPU1H5rlmVNBCYebL0x5jcg+QCr7t9nP5Yx5qCTRBljUoD/AVdblrXnn3Ms7mJVmCeGe4HHDhWvik4iIiIiIiIiIjWAZVlnHGydMSbTGJNiWVa6p6h0wDkcjDGxwI/A/ZZlza207z29pMqMMe8Cdx8uHg2vExERERERERGp+b4Drvb8fjXw7b4NjDFhwNfAfy3L+mKfdSme/xtgKLDicE+oopOIiIiIiIiISM03DhhgjFkPnOF5jDGmizHmLU+bi4DewDXGmKWenw6edR8aY/4E/gQSgCcO94QaXiciIiIiIiIiUsNZlpUL9D/A8oXADZ7fPwA+OMj2/Y72OVV0EhEREREREZEa5aAzZEtAaXidiIiIiIiIiIj4nIpOIiIiIiIiIiLicyo6iYiIiIiIiIiIz2lOJxERERERERGpUVzBDkAA9XQSERERERERERE/UNFJRERERERERER8TkUnERERERERERHxOc3pJCIiIiIiIiI1issEOwIB9XQSERERERERERE/UNFJRERERERERER8TkUnERERERERERHxOc3pJCIiIiIiIiI1igsr2CEI6ukkIiIiIiIiIiJ+oKKTiIiIiIiIiIj4nIpOIiIiIiIiIiLicyo6iYiIiIiIiIiIz2kicRERERERERGpUTSNePWgnk4iIiIiIiIiIuJzKjqJiIiIiIiIiIjPqegkIiIiIiIiIiI+pzmdRERERERERKRGcQU7AAHU00lERERERERERPxARScREREREREREfE5FZ1ERERERERERMTnNKdTgBlj7gcuA5y4h5nebFnWvAA+fx/gbsuy/hWI5zvpyatJ6N8RV0kZf458jYI/N+/XJrZdE9q+dAu2iDBypixh9f3vA9D83ouoN6gzlsuiPKeAP0e+RlnmTqKbp9L2xWHEtm3Cuqc/ZfNrPwQilSPS+smrSezfEWdJGcsPkW97T77ZU5awypNvC0++uCzKcgpY7skXIK57a1o/fhUmxE55XiHz/v1YINM6oOPt2P6TfFs9dDmJZ3bCqnBQvDmTP29/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k2n0wp2cNOMNHPmFbLntWQBy/juJE/5zO60mv4IxkPv5FErXbAag8etjCKkbg1XhZPtDr+MsCM48P5bTxbqx79Dhk/sxdhtpH09j99rtNBl9EYXLNpLzyyLSP5pK61duo9vcl3DkF7Hi5gkA7F67nazv5tBt5vO4HC7Wjnnb3avnEFqNv5Gy7dl0/vFJALJ/nMfm57/0d5pe/si39imtSLnodIpWbaHrlPEA/PXUx+ROWULzh66gVpvGYFmUbMtm7d0TA5brP3HPw+NYsGQ5+fkF9B96BbdefyXnDzk+hq8f81xOCl+aQN3x/wGbjdKfJuHcvJnoa6/DsXYNZbNnU2vYMExkJLUfedS9SWYW+Q/cBy4XRa+9Rt3nXgBjcKxbS8kP1etbgMF/77cbnv+Kdi/dQq/p48EY1jz+UcA/5PHHNSqsXl1avzQcY7eBzZD17RxyJy8GYMtL39D61ZE0vPlsnLtLWXPnGwHNV+RoGauGzcUgvvFzvUuOmxOj5szIcWTUvbHmOt7O5eNJ+6aB/dQ2mLLTagU7hIDKK40IdgjiJ71Wjgt2CAEVmhD44eHBktn39GCHEDCLVh3784gejYjj7G6qX+ZnJtgx+NPoxpfWyL9px2/++Jg6burpJCIiIiIiIiI1yvFVQqy+1OlBRERERERERER8TkUnERERERERERHxORWdRERERERERETE5zSnk4iIiIiIiIjUKC5q5Dzixxz1dBIREREREREREZ9T0UlERERERERERHxORScREREREREREfE5zekkIiIiIiIiIjWKZnSqHtTTSUREREREREREfE5FJxERERERERER8TkVnURERERERERExOc0p5OIiIiIiIiI1CiuYAcggHo6iYiIiIiIiIiIH6joJCIiIiIiIiIiPqeik4iIiIiIiIiI+JyKTiIiIiIiIiIi4nOaSFxEREREREREahQLK9ghCOrpJCIiIiIiIiIifqCik4iIiIiIiIiI+JyKTiIiIiIiIiIi4nOa00lEREREREREahRXsAMQQD2dRERERERERETED1R0EhERERERERERn1PRSUREREREREREfE5zOomIiIiIiIhIjeLCCnYIgno6iYiIiIiIiIiIH6joJCIiIiIiIiIiPqeik4iIiIiIiIiI+JzmdBIRERERERGRGkUzOlUP6ukkIiIiIiIiIiI+p6KTiIiIiIiIiIj4nIpOckCDMj8JdggBM/g4yhWOr2N7POUKx9e5fDzlClB/ztRghxAwHbZ8F+wQAqpf5mfBDiFgjqdcAUITmgY7hIA5nnIFqDdtRrBDCJjj7f32eLpOHU+5SnAZy9JIRzkgnRgiIiIiIiI1lwl2AP50S+OLauTftK9t/uyYOm6aSFxEREREREREahSX+lFUCxpeJyIiIiIiIiIiPqeik4iIiIiIiIiI+JyKTiIiIiIiIiIi4nOa00lEREREREREahRXsAMQQD2dRERERERERETED1R0EhERERERERERn1PRSUREREREREREfE5zOomIiIiIiIhIjWJhBTsEQT2dRERERERERETED1R0EhERERERERERn1PRSUREREREREREfE5zOomIiIiIiIhIjeIKdgACqKeTiIiIiIiIiIj4gYpOIiIiIiIiIiLicyo6iYiIiIiIiIiIz6noJCIiIiIiIiIiPqeJxEVERERERESkRrGwgh2CoJ5OIiIiIiIiIiLiByo6iYiIiIiIiIiIz6noJCIiIiIiIiIiPqc5nURERERERESkRnEFOwAB1NNJRERERERERET8QEUnERERERERERHxORWdRERERERERETE5zSnk4iIiIiIiIjUKC7LCnYIgno6iYiIiIiIiIiIH6joJCIiIiIiIiIiPqeik4iIiIiIiIiI+JzmdBIRERERERGRGkUzOlUP6ukUQMaYScaYOn9ju+nGmC5+CElERERERERExC/U0ylAjDEG+JdlWa5gx7KHMcZuWZYz2HGIiIiIiIiISM2jnk5+ZIxpbIxZa4z5L7ACcBpjEowx0caYH40xy4wxK4wxF3vadzbGzDDGLDLG/GKMSam0uwuNMfONMeuMMb087SOMMe8aY/40xiwxxvT1LL/GGPNKpTh+MMb08fxeZIx5zhizDDgtMP8SIiIiIiIiInK8UU8n/2sBXG1Z1lxjzGbPskFAmmVZZwMYY2obY0KBl4FzLcvK9hSingSu82wTYlnWKcaYwcDDwBnAcMCyLKutMeZE4FdjTMvDxBMNzLMs6y5fJikiIiIiIiIiUpl6OvnfFsuy5u6z7E9ggDHmGWNML8uydgGtgDbAZGPMUuABoEGlbb7y/H8R0Njze0/gAwDLstYAW4DDFZ2cwJcHWmGMuckYs9AYs3DixIlHkpuIiIiIiIhItePCqpE/xxr1dPK/3fsusCxrnTGmEzAYeMIYMwX4GlhpWdbBhryVef7v5PDHzUHVgmJEpd9LDzaPk2VZE4E91aZj72wWERERERERkWpDPZ2CwBiTChRblvUB8CzQCVgLJBpjTvO0CTXGnHyYXc0ELve0bwmc4NnPZqCDMcZmjGkInOKXREREREREREREDkI9nYKjLfCsMcYFVAC3WJZVboy5AHjJGFMb97GZAKw8xH5eBV4zxvyJu3fTNZZllRlj/gA2AauA1cBi/6UiIiIiIiIiIrI/Y1kaRSUHpBNDRERERESk5jLBDsCfLm00tEb+Tfvxlm+OqeOm4XUiIiIiIiIiIuJzKjqJiIiIiIiIiIjPqegkIiIiIiIiIiI+p4nERURERERERKRGcQU7AAHU00lERERERERERPxARScREREREREREfE5FZ1ERERERERERMTnNKeTiIiIiIiIiNQoLqxghyCop5OIiIiIiIiIiPiBik4iIiIiIiIiIuJzKjqJiIiIiIiIiIjPqegkIiIiIiIiIiI+p4nERURERERERKRGsTSReLWgnk4iIiIiIiIiIuJzKjqJiIiIiIiIiIjPqegkIiIiIiIiIiI+pzmdRERERERERKRGcQU7AAHU00lERERERERERPxARScREREREREREfE5FZ1ERERERERERMTnNKeTiIiIiIiIiNQolmUFOwRBPZ1ERERERERERMQPVHQSERERERERERGfU9FJRERERERERER8TnM6iYiIiIiIiEiN4kJzOlUH6ukkIiIiIiIiIiI+p6KTiIiIiIiIiIj4nIpOIiIiIiIiIiLicyo6iYiIiIiIiIiIz2kicRERERERERGpUVzBDqAaMsbEAZ8CjYHNwEWWZe08QDsn8Kfn4VbLss7xLG8CfALEA4uAKy3LKj/Uc6qnk4iIiIiIiIhIzTcGmGJZVgtgiufxgZRYltXB83NOpeXPAC9YltUc2Alcf7gnVE8nOaBPUy4PdggBMzX8kIVZOYadX2IPdggB1SI5N9ghBIzlMsEOIaAaTHo22CEEzK7rRgY7hIBavjw52CEEjDnOvrr65NaZwQ4hoOpNmxHsEAKmIuevYIcQMI1bDAl2CAG1/uGewQ4hoKJGvRHsECTwzgX6eH5/H5gO3HskGxpjDNAPuKzS9o8Arx1qO/V0EhERERERERE5BhhjbjLGLKz0c9NRbF7Psqx0z+8ZQL2DtIvw7HuuMWaoZ1k8kG9ZlsPzeDtQ/3BPqJ5OIiIiIiIiIlKjWDW0h61lWROBiQdbb4z5DThQV+r799mPZYw52D9SI8uydhhjmgJTjTF/Arv+TrwqOomIiIiIiIiI1ACWZZ1xsHXGmExjTIplWenGmBQg6yD72OH5/1/GmOlAR+BLoI4xJsTT26kBsONw8Wh4nYiIiIiIiIhIzfcdcLXn96uBb/dtYIypa4wJ9/yeAPQAVlmWZQHTgAsOtf2+VHQSEREREREREan5xgEDjDHrgTM8jzHGdDHGvOVpcxKw0BizDHeRaZxlWas86+4F7jTGbMA9x9Pbh3tCDa8TERERERERkRrFVUPndPonLMvKBfofYPlC4AbP77OBtgfZ/i/glKN5TvV0EhERERERERERn1PRSUREREREREREfE5FJxERERERERER8TnN6SQiIiIiIiIiNYr7y9Yk2NTTSUREREREREREfE5FJxERERERERER8TkVnURERERERERExOdUdBIREREREREREZ/TROIiIiIiIiIiUqO4gh2AAOrpJCIiIiIiIiIifqCik4iIiIiIiIiI+JyKTiIiIiIiIiIi4nOa00lEREREREREahQLK9ghCOrpJCIiIiIiIiIifqCik4iIiIiIiIiI+JyKTiIiIiIiIiIi4nOa00lEREREREREahSX5nSqFtTTSUREREREREREfE5FJxERERERERER8TkVnURERERERERExOc0p5OIiIiIiIiI1CiWpTmdqgP1dBIREREREREREZ9T0UlERERERERERHxORScREREREREREfE5FZ1ERERERERERMTnNJH4ETLGTAIusywr30/7n21ZVve/uW0foNyyrNk+DcoHOj5+FSn92+MsKWf+qDfY+efm/drUbdeYUyYMwx4RSvqUZSx58L8A1Gl9Ap2fuY6Q6Ah2b8tm7vBXcRSVENehKV2evQEAY2DFc1+x46eFgUzriFz88LW06duJ8pIy3rv7/9i2ctN+bUa+fz+xSXWw2+2sX7Cajx98G8vlokHrxlz+5I2Ehofhcjj56MG32LxsQxCyOHI1Pd9WT15NYv+OOEvKWDHyNQoPcC7HtGtCm5duwR4RRvaUJay9/30AWj50OYlndsJV4aB4cyYrb38dR0Gxd7uI+vF0n/kcG5/9gi2v/RColA4r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dBIREREREREREZ9T0UlERERERERERHxORScREREREREREfE5TSQuIiIiIiIiIjWKS1OJVwvq6SQiIiIiIiIiIj6nopOIiIiIiIiIiPicik4iIiIiIiIiIuJzmtNJRERERERERGoUzelUPaink4iIiIiIiIiI+JyKTiIiIiIiIiIi4nMqOomIiIiIiIiIiM9pTicRERERERERqVEsS3M6VQfq6SQiIiIiIiIiIj6nopOIiIiIiIiIiPicik4iIiIiIiIiIuJzmtNJRERERERERGoUF5rTqTpQTycREREREREREfE5FZ1ERERERERERMTnVHQSERERERERERGfU9FJRERERERERER8ThOJi4iIiIiIiEiNYmki8WpBPZ1ERERERERERMTnVHQSERERERERERGfU9FJRERERERERER8TnM6iYiIiIiIiEiNYlma06k6UE8nERERERERERHxOfV0kgNqbEqCHULAPNU2N9ghBNTOjRHBDiFgXJEm2CEE1KKspGCHEDC59uPr2N5/8oXBDiFgRtbtGuwQAqo35cEOIWBcHF+v20WrUoIdQkANDnYAAdS4xZBghxAwm9d/H+wQAioytVewQwgox6hgRyDHA/V0EhERERERERERn1NPJxERERERERGpUVxoTqfqQD2dRERERERERETE51R0EhERERERERERn1PRSUREREREREREfE5zOomIiIiIiIhIjWJZmtOpOlBPJxERERERERER8TkVnURERERERERExOdUdBIREREREREREZ9T0UlERERERERERHxOE4mLiIiIiIiISI3iQhOJVwfq6SQiIiIiIiIiIj6nopOIiIiIiIiIiPicik4iIiIiIiIiIuJzmtNJRERERERERGoUS3M6VQvq6SQiIiIiIiIiIj6nopOIiIiIiIiIiPicik4iIiIiIiIiIuJzmtNJRERERERERGoUl6U5naoD9XQSERERERERERGfU9FJRERERERERER8TkUnERERERERERHxOc3pJCIiIiIiIiI1ioXmdKoO1NNJRERERERERER8TkUnERERERERERHxORWdRERERERERETE51R0EhERERERERERn9NE4iIiIiIiIiJSo7gsTSReHaink4iIiIiIiIiI+JyKTiIiIiIiIiIi4nMqOomIiIiIiIiIiM9pTicRERERERERqVEsNKdTdaCi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iIiIiIiIiEjSKekkIiIiIiIiIiJJp6STiIiIiIiIiIgknXo6iYiIiIiIiEisuKunUyZQpZOIiIiIiIiISMyZWVsze93MZob/tqnnmGFmNjnhq8TMRob7HjWzbxL29dvU71TSSUREREREREQk/q4Exrr7dsDY8HYt7v6Wu/dz937AcGAdMCbhkMuq9rv75E39Qk2vk3otq2gcdQhp8+aU7lGHkFbd10cdgaTKjrlrog4hbfo0zq4/5HnXDok6hLRZ89rsqENIq6nTOkcdQtpYli1dXUBl1CFIisy8Lntek5t03TfqENKqeMG7UYcgkmpHAPuH3z8GjAOu+J7jjwFedfd1P/QXqtJJRERERERERGKl0j2WX2Z2tplNSPg6ewv+Wzq5+8Lw+0VAp00cfwLwZJ1tN5vZFDO73cw2Wa2iSicRERERERERkQbA3e8H7t/YfjN7A6ivlPqaOj/HzWyjZchm1gXYFRidsPkqgmRVfhjDFcCN3xevkk4iIiIiIiIiIjHg7gdsbJ+ZLTazLu6+MEwqLfmeH3Uc8Ly7V/e1SKiSKjWzR4BLNxWPpteJiIiIiIiIiMTfi8Avwu9/Afz3e449kTpT68JEFWZmwEhg2qZ+oSqdRERERERERCRWPMsWsNhMfwD+Y2ZnAnMJqpkwswHAue5+Vnh7G6A78Had+z9uZh0AAyYD527qFyrpJCIiIiIiIiISc+6+HBhRz/YJwFkJt+cA3eo5bviW/k5NrxMRERERERERkaRT0klERERERERERJJOSScREREREREREUk69XQSERERERERkVhxVyPxTKBKJxERERERERERSTolnUREREREREREJOmUdBIRERERERERkaRTTycRERERERERiZVK1NMpE6jSSUREREREREREkk5JJxERERERERERSTolnUREREREREREJOnU00lEREREREREYsVdPZ0ygSqdREREREREREQk6ZR0EhERERERERGRpFPSSUREREREREREkk49nUREREREREQkVirV0ykjqNJJRERERERERESSTkknERERERERERFJOiWdREREREREREQk6ZR0EhERERERERGRpFMjcRERERERERGJFVcj8YygSicREREREREREUk6JZ1ERERERERERCTplHQSEREREREREZGkU08nEREREREREYmVStTTKRMo6ZRiZnYaMMbd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Gb64YSwukAzr7jgn6hDSpuXlL0UdQloVL8jshF2yNWrfK7O7zv9IzZv2jOUbz5p13zSox03T60REREREREREJOmUdBIRERERERERkaRT0klERERERERERJJOjcRFREREREREJFZcvQQzgiqdREREREREREQk6ZR0EhERERERERGRpFPSSUREREREREREkk49nUREREREREQkVipdPZ0ygSqdREREREREREQk6ZR0EhERERERERGRpFPSSUREREREREREkk49nUREREREREQkVlw9nTKCKp1ERERERERERCTplHQSEREREREREZGkU9JJRERERERERESSTj2dRERERERERCRWHPV0ygSqdBIRERERERERkaRT0klERERERERERJJOSScREREREREREUk6JZ1ERERERERERCTp1EhcRERERERERGLFXY3EM4EqnUREREREREREJOmUdBIRERERERERkaRT0klERERERERERJJOPZ1EREREREREJFbU0ykzqNJJRERERERERESSTkknERERERERERFJOiWdREREREREREQk6dTTSURERERERERiRR2dMoMqnUREREREREREJOmUdBIRERERERERkaRT0klERERERERERJLO3DXTUTKHmZ3t7vdHHUc6ZNNYIbvGm01jhewabzaNFbJrvNk0VtB44yybxgrZNd5sGitk13izaaySfVTpJJnm7KgDSKNsGitk13izaayQXePNprFCdo03m8YKGm+cZdNYIbvGm01jhewabzaNVbKMkk4iIiIiIiIiIpJ0SjqJiIiIiIiIiEjSKekkmSab5jJn01ghu8abTWOF7BpvNo0Vsmu82TRW0HjjLJvGCtk13mwaK2TXeLNprJJl1EhcRERERERERESSTpVOIiIiIiIiIiKSdEo6iYiIiIiIiIhI0inpJJJGZra1mR0Qft/EzFpEHZOIiIiIiIhIKijpJJImZvZL4BngvnDTVsALkQWUJmFybYeo40g1M8uNOoZ0M7Nm2ThuiQ8zO9zMsuJcyMxyzezLqOMQ+THMLMfMBkcdR7qEz9vHo44jHSzQPeo4RCT58qIOQAQgPIHYhoS/SXf/R2QBpcYFwJ7AxwDuPtPMOkYbUmqZ2eHAn4F8oKeZ9QNudPefRRpYasw0s2eBR9z986iDSYXww/kJwMnAQKAUaGxmy4CXgfvcfVaE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iGyug4HTgK2Av1CTdCoEro4oJhERERERyVDq6SQiIlvEzI5292ejjkNERERERDKbejqJiMiW2sPMWlfdMLM2ZnZThPGIiIiIiEgGUtJJRES21KHuvqrqhruvBA6LLhwREREREclESjqJiMiWyjWzxlU3zKwJ0Ph7jhcRERERkSykRuIiIrKlHgfGmtkj4e3TgccijEdERERERDKQGomLiMgWM7NDgAPCm6+7++go4xERERERkcyjSicREfkhvgDK3f0NM2tqZi3cvSjqoEREREREJHOop5OIiGwRM/sl8AxwX7ipG/BCZAGJiIiIiEhGUtJJRES21AXAPkAhgLvPBDpGGpGIiIiIiGQcJZ1ERGRLlbp7WdUNM8sD1CBQRERERERqUdJJRES21NtmdjXQxMwOBJ4G/hdxTCIiIiIikmG0ep2IiGwRMzPgLOAgwIDRwIOuNxQREREREUmgpJOIiGw2M8sFprv7jlHHIiIiIiIimU3T60REZLO5ewUww8x6RB2LiIiIiIhktryoAxARkQanDTDdzD4B1lZtdPefRReSiIiIiIhkGiWdRERkS10bdQAiIiIiIpL51NNJRERERERERESSTpVOIiKyWczsPXcfYmZFQOIVCwPc3VtGFJqIiIiIiGQgVTqJiIiIiIiIiEjSqdJJRES2iJntCuwY3vzc3adHGY+IiIiIiGQmVTqJiMhmMbNWwH+BHsBnBNPqdgXmAUe4e2GE4YmIiIiISIZR0klERDaLmd0JlAGXu3tluC0H+APQxN0vjDI+ERERERHJLEo6iYjIZjGzz4Hd3L28zvY8YKq77xRNZCIiIiIikolyog5AREQajLK6CSeAcFtpBPGIiIiIiEgGUyNxERHZXAVmtjtBL6dEBjSOIB4REREREclgml4nIiKbxczGARt903D3YemLRkREREREMp2STiIiIiIiIiIiknSaXiciIpvFzI76vv3u/ly6YhERERERkcynpJOIiGyuw8N/OwKDgTfD28OADwAlnUREREREpJqSTiIislnc/XQAMxsD9HH3heHtLsCjEYYmIiIiIiIZKCfqAEREpMHpXpVwCi0GekQVjIiIiIiIZCZVOomIyJYaa2ajgSfD28cDb0QYj4iIiIiIZCCtXiciIlssbCq+b3jzHXd/Psp4REREREQk8yjpJCIiIiIiIiIiSaeeTiIiskXM7Cgzm2lmq82s0MyKzKww6rhERERERCSzqNJJRES2iJnNAg539y+ijkVERERERDKXKp1ERGRLLVbCSURERERENkWVTiIiskXM7K9AZ+AFoLRqu7s/F1VMIiIiIiKSefKiDkBERBqclsA64KCEbQ4o6SQiIiIiItVU6SQiIiIiIiIiIkmnSicREdksZna5u//RzO4iqGyqxd1HRRCWiIiIiIhkKCWdRERkczU2sz2Bz4AywCKOR0REREREMpiSTiIisrlaAXcAOwFTgPeBD4AP3H1FhHGJiIiIiEgGUk8nERHZImaWDwwABgN7h1+r3L1PpIGJiIiIiEhGUaWTiIhsqSYEK9i1Cr8WAFMjjUhERERERDKOKp1ERGSzmNn9wM5AEfAx8BHwkbuvjDQwERERERHJSDlRByAiIg1GD6AxsAiYD3wHrIoyIBERERERyVyqdBIRkc1mZkZQ7TQ4/NoFWAF86O7XRRmbiIiIiIhkFiWdRERki5nZVsA+BImnnwLt3L11pEGJiIiIiEhGUdJJREQ2i5mNoqbCaT3wQcLXVHevjDA8ERERERHJMFq9TkRENtc2wNPAxe6+MOJYREREREQkw6nSSUREREREREREkk6r14mIiIiIiIiISNIp6SQiIiIiIiIiIkmnpJOIiIiIiIiIiCSdkk4iIiIiIiIiIpJ0SjqJiIiIiIiIiEjS/T/uuoBbERlbegAAAABJRU5ErkJggg==\n",
      "text/plain": [
       "<Figure size 1440x1440 with 2 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "# plot the correlation matrix using heatmap for clear understanding\n",
    "plt.figure(figsize=(20,20))\n",
    "sns.heatmap(df.corr(), annot=True)\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Feature Selection using SelectKBest Method"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "GFG article link to $chi^{2}$ test - https://www.geeksforgeeks.org/chi-square-test-for-feature-selection-mathematical-explanation/"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 160,
   "metadata": {},
   "outputs": [],
   "source": [
    "bestfeatures = SelectKBest(score_func = chi2, k = 10)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "It works on the target label but instead we are passsing continuous float values to it. So, we need to convert our data to label form and there are two methods as follows:\n",
    "- usign LabelEncoder\n",
    "- multiplying the data by 100 and converting it to int which can be treated as labels by the model"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 161,
   "metadata": {},
   "outputs": [],
   "source": [
    "# use the label encoder\n",
    "from sklearn import preprocessing\n",
    "label_encoder = preprocessing.LabelEncoder()\n",
    "train_Y = label_encoder.fit_transform(target)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 169,
   "metadata": {},
   "outputs": [],
   "source": [
    "target_cont = df['Radiation'].apply(lambda x : int(x*100))\n",
    "scaled_input_features = MinMaxScaler().fit_transform(input_features)\n",
    "fit = bestfeatures.fit(scaled_input_features, target_cont)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 170,
   "metadata": {},
   "outputs": [],
   "source": [
    "scores = pd.DataFrame(fit.scores_)\n",
    "column = pd.DataFrame(input_features.columns)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 171,
   "metadata": {},
   "outputs": [],
   "source": [
    "# contatinating data_features with the scores\n",
    "featureScores = pd.concat([column, scores], axis=1)\n",
    "\n",
    "#naming the dataframe columns\n",
    "featureScores.columns = ['Features', 'feature_imp'] "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 172,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Features</th>\n",
       "      <th>feature_imp</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>12</th>\n",
       "      <td>sethour</td>\n",
       "      <td>12207.531454</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>5</th>\n",
       "      <td>Month</td>\n",
       "      <td>4684.579610</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>11</th>\n",
       "      <td>riseminuter</td>\n",
       "      <td>4015.062771</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>WindDirection(Degrees)</td>\n",
       "      <td>3271.827277</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>6</th>\n",
       "      <td>Day</td>\n",
       "      <td>2841.926850</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>8</th>\n",
       "      <td>Minute</td>\n",
       "      <td>2702.449333</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>9</th>\n",
       "      <td>Second</td>\n",
       "      <td>2288.673032</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>13</th>\n",
       "      <td>setminute</td>\n",
       "      <td>1863.712087</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>Temperature</td>\n",
       "      <td>1651.685632</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>Humidity</td>\n",
       "      <td>1588.087433</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>Speed</td>\n",
       "      <td>765.859779</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>7</th>\n",
       "      <td>Hour</td>\n",
       "      <td>691.185393</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>Pressure</td>\n",
       "      <td>523.791060</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>10</th>\n",
       "      <td>risehour</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "                  Features   feature_imp\n",
       "12                 sethour  12207.531454\n",
       "5                    Month   4684.579610\n",
       "11             riseminuter   4015.062771\n",
       "3   WindDirection(Degrees)   3271.827277\n",
       "6                      Day   2841.926850\n",
       "8                   Minute   2702.449333\n",
       "9                   Second   2288.673032\n",
       "13               setminute   1863.712087\n",
       "0              Temperature   1651.685632\n",
       "2                 Humidity   1588.087433\n",
       "4                    Speed    765.859779\n",
       "7                     Hour    691.185393\n",
       "1                 Pressure    523.791060\n",
       "10                risehour           NaN"
      ]
     },
     "execution_count": 172,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# best features\n",
    "featureScores.sort_values(by = 'feature_imp', ascending=False, inplace=True)\n",
    "featureScores"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 174,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 720x432 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "# visualise the feature importance\n",
    "plt.figure(figsize = (10, 6))\n",
    "plt.bar(featureScores.Features, featureScores.feature_imp)\n",
    "plt.xticks(rotation = 70)\n",
    "plt.xlabel(\"Features\")\n",
    "plt.ylabel(\"Importance\")\n",
    "plt.title(\"Feature Importance using Extra Tree Classifier\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Feature Selection using Extra Tree Classifier"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 180,
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "[Parallel(n_jobs=1)]: Using backend SequentialBackend with 1 concurrent workers.\n"
     ]
    },
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "building tree 1 of 10\n"
     ]
    },
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "[Parallel(n_jobs=1)]: Done   1 out of   1 | elapsed:    6.1s remaining:    0.0s\n"
     ]
    },
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "building tree 2 of 10\n",
      "building tree 3 of 10\n",
      "building tree 4 of 10\n",
      "building tree 5 of 10\n",
      "building tree 6 of 10\n",
      "building tree 7 of 10\n",
      "building tree 8 of 10\n",
      "building tree 9 of 10\n",
      "building tree 10 of 10\n"
     ]
    },
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "[Parallel(n_jobs=1)]: Done  10 out of  10 | elapsed:  1.0min finished\n"
     ]
    },
    {
     "data": {
      "text/plain": [
       "ExtraTreesClassifier(n_estimators=10, verbose=2)"
      ]
     },
     "execution_count": 180,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "model = ExtraTreesClassifier(verbose = 2, n_estimators = 10)\n",
    "model.fit(scaled_input_features, target_cont)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 181,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>feature_imp</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>WindDirection(Degrees)</th>\n",
       "      <td>0.158561</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Minute</th>\n",
       "      <td>0.147512</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Speed</th>\n",
       "      <td>0.123968</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Second</th>\n",
       "      <td>0.123245</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Humidity</th>\n",
       "      <td>0.109846</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Hour</th>\n",
       "      <td>0.092982</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Temperature</th>\n",
       "      <td>0.082955</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Pressure</th>\n",
       "      <td>0.075890</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Day</th>\n",
       "      <td>0.034581</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>setminute</th>\n",
       "      <td>0.023260</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>riseminuter</th>\n",
       "      <td>0.022119</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Month</th>\n",
       "      <td>0.003720</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>sethour</th>\n",
       "      <td>0.001361</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>risehour</th>\n",
       "      <td>0.000000</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "                        feature_imp\n",
       "WindDirection(Degrees)     0.158561\n",
       "Minute                     0.147512\n",
       "Speed                      0.123968\n",
       "Second                     0.123245\n",
       "Humidity                   0.109846\n",
       "Hour                       0.092982\n",
       "Temperature                0.082955\n",
       "Pressure                   0.075890\n",
       "Day                        0.034581\n",
       "setminute                  0.023260\n",
       "riseminuter                0.022119\n",
       "Month                      0.003720\n",
       "sethour                    0.001361\n",
       "risehour                   0.000000"
      ]
     },
     "execution_count": 181,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "feature_importances = pd.DataFrame(model.feature_importances_, index = input_features.columns, columns = [\"feature_imp\"])\n",
    "feature_importances.sort_values(by = 'feature_imp', ascending=False, inplace = True)\n",
    "feature_importances"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 182,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 720x432 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "plt.figure(figsize = (10, 6))\n",
    "plt.bar(feature_importances.index, feature_importances[\"feature_imp\"])\n",
    "plt.xticks(rotation = 70)\n",
    "plt.xlabel(\"Features\")\n",
    "plt.ylabel(\"Importance\")\n",
    "plt.title(\"Feature Importance using Extra Tree Classifier\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Feature Engineering with BoxCox, Log, Min-Max and Standard transformation"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 206,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 720x360 with 5 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 720x360 with 5 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 720x360 with 5 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 720x360 with 5 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 720x360 with 5 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "# Looking for transformation\n",
    "features_to_transform = ['Temperature', 'Pressure', 'Humidity', 'Speed', 'WindDirection(Degrees)']\n",
    "\n",
    "for i in features_to_transform:\n",
    "    \n",
    "    fig, (ax1, ax2, ax3, ax4, ax5) = plt.subplots(5, 1, figsize=(10, 5))\n",
    "    \n",
    "    pd.DataFrame(input_features[i]).hist(ax = ax1, bins = 50)\n",
    "    pd.DataFrame((input_features[i]+1).transform(np.log)).hist(ax = ax2, bins = 50)\n",
    "    pd.DataFrame(stats.boxcox(input_features[i]+1)[0]).hist(ax = ax3, bins = 50)    \n",
    "    pd.DataFrame(StandardScaler().fit_transform(np.array(input_features[i]).reshape(-1, 1))).hist(ax = ax4, bins = 50)\n",
    "    pd.DataFrame(MinMaxScaler().fit_transform(np.array(input_features[i]).reshape(-1, 1))).hist(ax = ax5, bins = 50)\n",
    "    \n",
    "    ax1.set_ylabel('Normal')\n",
    "    ax2.set_ylabel('Log')\n",
    "    ax3.set_ylabel('Box Cox')\n",
    "    ax4.set_ylabel('Standard')\n",
    "    ax5.set_ylabel('MinMax')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 207,
   "metadata": {},
   "outputs": [],
   "source": [
    "# set the transformations required\n",
    "transform = {'Temperature' : (input_features['Temperature']+1).transform(np.log), \n",
    "             'Pressure': stats.boxcox(input_features['Pressure']+1)[0], \n",
    "            'Humidity' : stats.boxcox(input_features['Humidity']+1)[0], \n",
    "            'Speed' : (input_features['Speed']+1).transform(np.log), \n",
    "            'WindDirection(Degrees)' : MinMaxScaler().fit_transform(\n",
    "                np.array(input_features['WindDirection(Degrees)']).reshape(-1, 1))}\n",
    "\n",
    "for i in transform:\n",
    "    input_features[i] = transform[i]"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 208,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Temperature</th>\n",
       "      <th>Pressure</th>\n",
       "      <th>Humidity</th>\n",
       "      <th>WindDirection(Degrees)</th>\n",
       "      <th>Speed</th>\n",
       "      <th>Month</th>\n",
       "      <th>Day</th>\n",
       "      <th>Hour</th>\n",
       "      <th>Minute</th>\n",
       "      <th>Second</th>\n",
       "      <th>risehour</th>\n",
       "      <th>riseminuter</th>\n",
       "      <th>sethour</th>\n",
       "      <th>setminute</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>0.413264</td>\n",
       "      <td>5.044903e+152</td>\n",
       "      <td>1141.775867</td>\n",
       "      <td>0.492692</td>\n",
       "      <td>0.360847</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>22.145042</td>\n",
       "      <td>26</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>0.413264</td>\n",
       "      <td>5.044903e+152</td>\n",
       "      <td>1106.377133</td>\n",
       "      <td>0.490996</td>\n",
       "      <td>0.339319</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>20.661485</td>\n",
       "      <td>23</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>0.413264</td>\n",
       "      <td>5.044903e+152</td>\n",
       "      <td>1071.498314</td>\n",
       "      <td>0.440894</td>\n",
       "      <td>0.339319</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>19.132762</td>\n",
       "      <td>26</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>0.413264</td>\n",
       "      <td>5.044903e+152</td>\n",
       "      <td>1177.693409</td>\n",
       "      <td>0.382426</td>\n",
       "      <td>0.339319</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>17.552332</td>\n",
       "      <td>21</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>0.413264</td>\n",
       "      <td>5.044903e+152</td>\n",
       "      <td>1251.080579</td>\n",
       "      <td>0.291391</td>\n",
       "      <td>0.360847</td>\n",
       "      <td>9</td>\n",
       "      <td>29</td>\n",
       "      <td>23</td>\n",
       "      <td>15.911768</td>\n",
       "      <td>24</td>\n",
       "      <td>6</td>\n",
       "      <td>13</td>\n",
       "      <td>18</td>\n",
       "      <td>13</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   Temperature       Pressure     Humidity  WindDirection(Degrees)     Speed  \\\n",
       "0     0.413264  5.044903e+152  1141.775867                0.492692  0.360847   \n",
       "1     0.413264  5.044903e+152  1106.377133                0.490996  0.339319   \n",
       "2     0.413264  5.044903e+152  1071.498314                0.440894  0.339319   \n",
       "3     0.413264  5.044903e+152  1177.693409                0.382426  0.339319   \n",
       "4     0.413264  5.044903e+152  1251.080579                0.291391  0.360847   \n",
       "\n",
       "   Month  Day  Hour     Minute  Second  risehour  riseminuter  sethour  \\\n",
       "0      9   29    23  22.145042      26         6           13       18   \n",
       "1      9   29    23  20.661485      23         6           13       18   \n",
       "2      9   29    23  19.132762      26         6           13       18   \n",
       "3      9   29    23  17.552332      21         6           13       18   \n",
       "4      9   29    23  15.911768      24         6           13       18   \n",
       "\n",
       "   setminute  \n",
       "0         13  \n",
       "1         13  \n",
       "2         13  \n",
       "3         13  \n",
       "4         13  "
      ]
     },
     "execution_count": 208,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "input_features.head()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Preparing data - Standardisation and Splitting"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 227,
   "metadata": {},
   "outputs": [],
   "source": [
    "xtrain, xtest, ytrain, ytest = train_test_split(input_features, target, test_size=0.2, random_state=1)\n",
    "\n",
    "scaler = StandardScaler()\n",
    "xtrain = scaler.fit_transform(xtrain)\n",
    "xtest = scaler.transform(xtest)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 235,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "((26148, 14), (6538, 14))"
      ]
     },
     "execution_count": 235,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "xtrain.shape, xtest.shape"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Prediction with XGBoost"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 252,
   "metadata": {},
   "outputs": [],
   "source": [
    "# declare parameters\n",
    "params = {\n",
    "    'learning_rate': 0.1,\n",
    "    'max_depth': 8}\n",
    "\n",
    "from xgboost import XGBRegressor\n",
    "model = XGBRegressor(**params)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 253,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "XGBRegressor(base_score=0.5, booster='gbtree', colsample_bylevel=1,\n",
       "             colsample_bynode=1, colsample_bytree=1, enable_categorical=False,\n",
       "             gamma=0, gpu_id=-1, importance_type=None,\n",
       "             interaction_constraints='', learning_rate=0.1, max_delta_step=0,\n",
       "             max_depth=8, min_child_weight=1, missing=nan,\n",
       "             monotone_constraints='()', n_estimators=100, n_jobs=13,\n",
       "             num_parallel_tree=1, predictor='auto', random_state=0, reg_alpha=0,\n",
       "             reg_lambda=1, scale_pos_weight=1, subsample=1, tree_method='exact',\n",
       "             validate_parameters=1, verbosity=None)"
      ]
     },
     "execution_count": 253,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# train the model\n",
    "model.fit(xtrain, ytrain)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 256,
   "metadata": {},
   "outputs": [],
   "source": [
    "y_pred = model.predict(xtest)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 257,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "XGBoost model result: 81.4440\n"
     ]
    }
   ],
   "source": [
    "print('XGBoost model result: {0:0.4f}'. format(np.sqrt(mean_squared_error(ytest, y_pred))))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 258,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Testing performance\n",
      "RMSE: 81.44\n",
      "R2: 0.93\n"
     ]
    }
   ],
   "source": [
    "rmse = np.sqrt(mean_squared_error(ytest, y_pred))\n",
    "r2 = r2_score(ytest, y_pred)\n",
    "\n",
    "print(\"Testing performance\")\n",
    "\n",
    "print(\"RMSE: {:.2f}\".format(rmse))\n",
    "print(\"R2: {:.2f}\".format(r2))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Using MultiLayer Perceptron for prediction"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 295,
   "metadata": {},
   "outputs": [],
   "source": [
    "xtrain, xtest, ytrain, ytest = train_test_split(input_features, target, test_size=0.2, random_state=1)\n",
    "\n",
    "scaler = StandardScaler()\n",
    "xtrain = scaler.fit_transform(xtrain)\n",
    "xtest = scaler.transform(xtest)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 332,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Model: \"sequential_20\"\n",
      "_________________________________________________________________\n",
      "Layer (type)                 Output Shape              Param #   \n",
      "=================================================================\n",
      "dense_79 (Dense)             (None, 128)               1920      \n",
      "_________________________________________________________________\n",
      "dropout_59 (Dropout)         (None, 128)               0         \n",
      "_________________________________________________________________\n",
      "dense_80 (Dense)             (None, 64)                8256      \n",
      "_________________________________________________________________\n",
      "dropout_60 (Dropout)         (None, 64)                0         \n",
      "_________________________________________________________________\n",
      "dense_81 (Dense)             (None, 32)                2080      \n",
      "_________________________________________________________________\n",
      "dropout_61 (Dropout)         (None, 32)                0         \n",
      "_________________________________________________________________\n",
      "dense_82 (Dense)             (None, 1)                 33        \n",
      "=================================================================\n",
      "Total params: 12,289\n",
      "Trainable params: 12,289\n",
      "Non-trainable params: 0\n",
      "_________________________________________________________________\n",
      "None\n"
     ]
    }
   ],
   "source": [
    "model = None\n",
    "model = Sequential()\n",
    "    \n",
    "model.add(Dense(128, activation='relu', input_dim=14))\n",
    "model.add(Dropout(0.33))\n",
    "    \n",
    "model.add(Dense(64, activation='relu'))\n",
    "model.add(Dropout(0.33))\n",
    "\n",
    "model.add(Dense(32, activation='relu'))\n",
    "model.add(Dropout(0.33))\n",
    "\n",
    "model.add(Dense(1, activation='linear'))\n",
    "    \n",
    "model.compile(metrics='mse', loss='mae', optimizer=Adam(learning_rate=0.001))\n",
    "print(model.summary())"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 334,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Epoch 1/50\n",
      "736/736 [==============================] - 1s 1ms/step - loss: 77.0724 - mse: 22450.2949 - val_loss: 60.5145 - val_mse: 14623.6143\n",
      "Epoch 2/50\n",
      "736/736 [==============================] - 1s 925us/step - loss: 73.7126 - mse: 21135.0723 - val_loss: 57.0598 - val_mse: 13359.2012\n",
      "Epoch 3/50\n",
      "736/736 [==============================] - 1s 866us/step - loss: 73.4762 - mse: 20975.4980 - val_loss: 56.2689 - val_mse: 13153.0127\n",
      "Epoch 4/50\n",
      "736/736 [==============================] - 1s 922us/step - loss: 70.8537 - mse: 19687.3379 - val_loss: 56.5140 - val_mse: 13439.2090\n",
      "Epoch 5/50\n",
      "736/736 [==============================] - 1s 891us/step - loss: 70.5504 - mse: 19798.8203 - val_loss: 52.0351 - val_mse: 11911.0508\n",
      "Epoch 6/50\n",
      "736/736 [==============================] - 1s 942us/step - loss: 69.6108 - mse: 19411.9590 - val_loss: 51.3607 - val_mse: 11713.5078\n",
      "Epoch 7/50\n",
      "736/736 [==============================] - 1s 939us/step - loss: 68.3939 - mse: 18717.3340 - val_loss: 50.1383 - val_mse: 11289.9922\n",
      "Epoch 8/50\n",
      "736/736 [==============================] - 1s 933us/step - loss: 68.3117 - mse: 18774.4414 - val_loss: 51.1353 - val_mse: 11579.8125\n",
      "Epoch 9/50\n",
      "736/736 [==============================] - 1s 877us/step - loss: 67.3591 - mse: 18315.0879 - val_loss: 52.8519 - val_mse: 12205.6240\n",
      "Epoch 10/50\n",
      "736/736 [==============================] - 1s 848us/step - loss: 67.1457 - mse: 18158.3535 - val_loss: 51.7748 - val_mse: 11889.8359\n",
      "Epoch 11/50\n",
      "736/736 [==============================] - 1s 879us/step - loss: 66.4651 - mse: 17934.0957 - val_loss: 49.7666 - val_mse: 11265.1025\n",
      "Epoch 12/50\n",
      "736/736 [==============================] - 1s 963us/step - loss: 66.1736 - mse: 17871.7988 - val_loss: 49.8094 - val_mse: 11456.9746\n",
      "Epoch 13/50\n",
      "736/736 [==============================] - 1s 1ms/step - loss: 65.4015 - mse: 17486.2637 - val_loss: 48.6689 - val_mse: 11157.5645\n",
      "Epoch 14/50\n",
      "736/736 [==============================] - 1s 925us/step - loss: 65.5265 - mse: 17615.4434 - val_loss: 48.5563 - val_mse: 10851.7666\n",
      "Epoch 15/50\n",
      "736/736 [==============================] - 1s 869us/step - loss: 64.8271 - mse: 17450.1055 - val_loss: 50.4920 - val_mse: 11390.1992\n",
      "Epoch 16/50\n",
      "736/736 [==============================] - 1s 877us/step - loss: 64.3580 - mse: 17148.0586 - val_loss: 48.5519 - val_mse: 10592.4404\n",
      "Epoch 17/50\n",
      "736/736 [==============================] - 1s 827us/step - loss: 64.3280 - mse: 17154.2012 - val_loss: 47.5486 - val_mse: 10522.7412\n",
      "Epoch 18/50\n",
      "736/736 [==============================] - 1s 868us/step - loss: 63.7460 - mse: 16896.2480 - val_loss: 47.2190 - val_mse: 10462.2305\n",
      "Epoch 19/50\n",
      "736/736 [==============================] - 1s 881us/step - loss: 63.5047 - mse: 16785.1992 - val_loss: 44.1539 - val_mse: 9901.5293\n",
      "Epoch 20/50\n",
      "736/736 [==============================] - 1s 945us/step - loss: 63.5797 - mse: 16945.1094 - val_loss: 44.9712 - val_mse: 10014.4805\n",
      "Epoch 21/50\n",
      "736/736 [==============================] - 1s 932us/step - loss: 63.9419 - mse: 16964.1289 - val_loss: 47.5097 - val_mse: 10712.3369\n",
      "Epoch 22/50\n",
      "736/736 [==============================] - 1s 882us/step - loss: 63.0819 - mse: 16628.7578 - val_loss: 45.0204 - val_mse: 9953.0527\n",
      "Epoch 23/50\n",
      "736/736 [==============================] - 1s 853us/step - loss: 63.6231 - mse: 16859.9219 - val_loss: 46.8019 - val_mse: 10686.9990\n",
      "Epoch 24/50\n",
      "736/736 [==============================] - 1s 859us/step - loss: 63.0761 - mse: 16729.6875 - val_loss: 47.0547 - val_mse: 10399.9805\n",
      "Epoch 25/50\n",
      "736/736 [==============================] - 1s 901us/step - loss: 62.1768 - mse: 16470.2090 - val_loss: 48.3749 - val_mse: 10694.1680\n",
      "Epoch 26/50\n",
      "736/736 [==============================] - 1s 882us/step - loss: 62.2230 - mse: 16158.0469 - val_loss: 49.2070 - val_mse: 10972.6016\n",
      "Epoch 27/50\n",
      "736/736 [==============================] - 1s 912us/step - loss: 62.4079 - mse: 16420.7305 - val_loss: 47.6120 - val_mse: 10522.3164\n",
      "Epoch 28/50\n",
      "736/736 [==============================] - 1s 896us/step - loss: 61.7679 - mse: 16064.8809 - val_loss: 44.8617 - val_mse: 10127.3809\n",
      "Epoch 29/50\n",
      "736/736 [==============================] - 1s 878us/step - loss: 62.0615 - mse: 16339.1650 - val_loss: 46.1067 - val_mse: 10199.7793\n",
      "Epoch 30/50\n",
      "736/736 [==============================] - 1s 897us/step - loss: 61.0412 - mse: 15766.6729 - val_loss: 46.2059 - val_mse: 10298.5684\n",
      "Epoch 31/50\n",
      "736/736 [==============================] - 1s 875us/step - loss: 62.2449 - mse: 16476.5820 - val_loss: 48.8263 - val_mse: 11009.4795\n",
      "Epoch 32/50\n",
      "736/736 [==============================] - 1s 933us/step - loss: 62.0014 - mse: 16174.3662 - val_loss: 48.4816 - val_mse: 10910.6553\n",
      "Epoch 33/50\n",
      "736/736 [==============================] - 1s 920us/step - loss: 61.4542 - mse: 16024.5303 - val_loss: 46.2172 - val_mse: 10211.0703\n",
      "Epoch 34/50\n",
      "736/736 [==============================] - 1s 861us/step - loss: 61.3468 - mse: 16157.6191 - val_loss: 45.7437 - val_mse: 10247.7158\n",
      "Epoch 35/50\n",
      "736/736 [==============================] - 1s 1ms/step - loss: 61.1599 - mse: 16101.3994 - val_loss: 47.8036 - val_mse: 10748.8135\n",
      "Epoch 36/50\n",
      "736/736 [==============================] - 1s 941us/step - loss: 60.2658 - mse: 15724.0996 - val_loss: 43.5047 - val_mse: 9715.8984\n",
      "Epoch 37/50\n",
      "736/736 [==============================] - 1s 1ms/step - loss: 60.0708 - mse: 15448.2598 - val_loss: 46.2766 - val_mse: 10014.5908\n",
      "Epoch 38/50\n",
      "736/736 [==============================] - 1s 908us/step - loss: 60.5259 - mse: 15660.7754 - val_loss: 45.8366 - val_mse: 10344.3135\n",
      "Epoch 39/50\n",
      "736/736 [==============================] - 1s 873us/step - loss: 60.4888 - mse: 15904.0791 - val_loss: 43.4116 - val_mse: 10010.5625\n",
      "Epoch 40/50\n",
      "736/736 [==============================] - 1s 861us/step - loss: 60.3809 - mse: 15694.5420 - val_loss: 46.7645 - val_mse: 10100.0537\n",
      "Epoch 41/50\n",
      "736/736 [==============================] - 1s 978us/step - loss: 60.1293 - mse: 15513.8223 - val_loss: 44.0342 - val_mse: 9818.5117\n",
      "Epoch 42/50\n",
      "736/736 [==============================] - 1s 970us/step - loss: 59.4724 - mse: 15323.5449 - val_loss: 49.2446 - val_mse: 10788.7910\n",
      "Epoch 43/50\n",
      "736/736 [==============================] - 1s 883us/step - loss: 59.3943 - mse: 15095.6211 - val_loss: 45.3726 - val_mse: 9744.9512\n",
      "Epoch 44/50\n",
      "736/736 [==============================] - 1s 880us/step - loss: 60.2631 - mse: 15723.0322 - val_loss: 45.8757 - val_mse: 9736.2402\n",
      "Epoch 45/50\n",
      "736/736 [==============================] - 1s 988us/step - loss: 60.1736 - mse: 15591.7822 - val_loss: 42.2669 - val_mse: 9189.6729\n",
      "Epoch 46/50\n",
      "736/736 [==============================] - 1s 1ms/step - loss: 59.6529 - mse: 15329.9092 - val_loss: 48.0413 - val_mse: 10367.2129\n",
      "Epoch 47/50\n",
      "736/736 [==============================] - 1s 913us/step - loss: 59.8662 - mse: 15613.3496 - val_loss: 44.7841 - val_mse: 9852.1357\n",
      "Epoch 48/50\n",
      "736/736 [==============================] - 1s 978us/step - loss: 60.0003 - mse: 15500.7832 - val_loss: 42.2743 - val_mse: 9482.4902\n",
      "Epoch 49/50\n",
      "736/736 [==============================] - 1s 881us/step - loss: 60.4635 - mse: 15654.1611 - val_loss: 42.7643 - val_mse: 9595.6260\n",
      "Epoch 50/50\n",
      "736/736 [==============================] - 1s 919us/step - loss: 60.5116 - mse: 15704.3730 - val_loss: 42.9297 - val_mse: 9528.4600\n"
     ]
    }
   ],
   "source": [
    "history = model.fit(xtrain, ytrain, validation_split=0.1, epochs=50, batch_size=32)\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 335,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "fit = history.history\n",
    "for i in fit:\n",
    "    plt.plot(fit[i])\n",
    "    plt.title(i + ' over epochs')\n",
    "    plt.ylabel(i)\n",
    "    plt.xlabel('epochs')\n",
    "    plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 336,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "205/205 [==============================] - 0s 643us/step - loss: 40.6781 - mse: 9589.2979\n",
      "Mean absolute error:  40.67810821533203\n"
     ]
    }
   ],
   "source": [
    "scores = model.evaluate(xtest, ytest)\n",
    "mae = scores[0]\n",
    "mse = scores[1]\n",
    "print('Mean absolute error: ', mae)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 339,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "40.678090141121494"
      ]
     },
     "execution_count": 339,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "mean_absolute_error(ytest, model.predict(xtest))"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3 (ipykernel)",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.8.13"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 4
}
