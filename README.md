# PowerBI DAX and Power Query
Hints, formulas and scripts in Power BI (DAX and Power Query M)
## Dax
# 15 days moving average

Moving AverageX 7 Days = 

AVERAGEX (
    
    DATESINPERIOD (
    
        'DIM Timeseries'[Date],
        
        LASTDATE ( 'DIM Timeseries'[Date] ),
        
        -15,
        
        DAY
        
    ),
    
    SUM(Daily[HVAC])
    )


# Corresponding Dates last year

AvgTempPY = CALCULATE(AVERAGE(Daily[AvgTemp]), SAMEPERIODLASTYEAR('DIM Timeseries'[Date]))

# Remove ZERO (or Blank):

Performance compared to last year same period = IF (FactPower[Total kWh LY]>0, ( 1-(DIVIDE(SUM(FactPower[Total_kWH]), [Total kWh LY]))))

# Or for blank

IF NOT(ISBLANK((FactPower[Total kWh LY]>0, ( 1-(DIVIDE(SUM(FactPower[Total_kWH]), [Total kWh LY]))))

# SUM or Average from a specific date to today or a specific date

Main Meter 2022 (From Completion Date) = CALCULATE(

        AVERAGE(Daily[Main])
        
        ,FILTER(Daily,
        
        (Daily[Date] >= Daily[Completion Date] && Daily[Date] < DATE(2022,08,12))
        
        ))
        
# And for the last year

Main Meter 2021 (From Completion Date) = CALCULATE(

        AVERAGE(Daily[Main])
        
        ,FILTER(Daily,
        
        (Daily[Date] >= Daily[Completion Date PY] && Daily[Date] < DATE(2021,08,12))
        
        ))
        
# Year over Year change
Main YoY% = 

VAR __PREV_YEAR = CALCULATE(SUM('Daily'[Main]), DATEADD('DIM Timeseries'[Date], -1, YEAR))

RETURN

    1-(DIVIDE(SUM('Daily'[Main]), __PREV_YEAR))

# Month over month change

%MoM = 

VAR __PREV_MONTH =

    CALCULATE(
    
        SUM('Fact Readings (dataflow)'[Consumption (kWh)]),
        
        DATEADD('DimDates'[Date], -1, MONTH)
        
    )
    
RETURN

    DIVIDE(
    
        SUM('Fact Readings (dataflow)'[Consumption (kWh)])
        
            - __PREV_MONTH,
            
        __PREV_MONTH
        
    )
    
Average of consumption same weekdays (historical) = 

VAR Todaysname= WEEKDAY(TODAY(), 2)

Return

CALCULATE(

    SUM 
    
    ('Fact Readings (dataflow)'[Consumption (kWh)]), FILTER(DimDates, DimDates[Day of Week Number]=Todaysname))
   


# Change Total Same Day Last Week % difference from Total Today = 

VAR __BASELINE_VALUE = [Total Today]

VAR __VALUE_TO_COMPARE = [Total Same Day Last Week]

RETURN
IF(

        NOT ISBLANK(__VALUE_TO_COMPARE),
        
    DIVIDE(__VALUE_TO_COMPARE - __BASELINE_VALUE, __BASELINE_VALUE))
    


# One-month-before a date to that date

Test_General 2022 (one month before Completion Date) = CALCULATE(

        AVERAGE(Daily[General])
        
        ,FILTER(Daily,
        
        (Daily[Date] < Daily[Completion Date] && Daily[Date] > EDATE(Daily[Completion Date], -1)
        
        )))

# And Last year same period

TEST_General 2021 (one month before Completion Date) = CALCULATE(

        AVERAGE(Daily[General])
        
        ,FILTER(Daily,
        
        (Daily[Date] < Daily[Completion Date PY] && Daily[Date] > EDATE(Daily[Completion Date PY], -1)
        
        )))
        
# Compare a value in two different time periods:

First, create a duplication of date table: Comparison Date; makes it relationship to your intended value INACTIVE.
Second: two time slicers from the two date tables.

# Your comparison Measure: 

Total Comparison = CALCULATE([Total Consumption], ALL(DimDates), USERELATIONSHIP('Comparison Date'[Date], 'Fact Readings (dataflow)'[Date]))


# Total Consumption = 

CALCULATE(

    SUM('Fact Readings (dataflow)'[Consumption (kWh)]),
    
    ALLSELECTED('DimDates'[Date])
    
)


# % difference of base value to compared value = 

VAR __BASELINE_VALUE = [Total Consumption]

VAR __VALUE_TO_COMPARE = [Total Comparison]

RETURN

    DIVIDE(__VALUE_TO_COMPARE - __BASELINE_VALUE, __BASELINE_VALUE)
    

# Dynamic TITLE

Title = 

VAR BaseDate= SELECTEDVALUE(DimDates[Month-Year])

VAR ComparisonDate= SELECTEDVALUE('Comparison Date'[Month-Year])

VAR Result= "Consumption in " & BaseDate & " compared to " & ComparisonDate

Return Result

#Date and Time functions

Today = NOW()

LastMonth = MONTH(EOMONTH(TODAY(),-1))

Today's Day Name = FORMAT([Today],"dddd")

Yesterday = TODAY()-1

Month Last 30 Days = CALCULATE(

    SUM('Fact Readings (dataflow)'[Consumption (kWh)]), DATESINPERIOD(DimDates[Date], TODAY(), -1, MONTH))
    
Month to Date (Calendar) = CALCULATE (SUM ( 'Fact Readings (dataflow)'[Consumption (kWh)] ), DimDates[Month Offset] = 0)

Month to date previous = 

TOTALMTD(SUM('Fact Readings (dataflow)'[Consumption (kWh)]),DATEADD(DimDates[Date],-1,MONTH ))


# Dynamic title example: Last Communication (ago) = 

Last Communication (meter) = MAX('Fact Readings (dataflow)'[NewDateTime])

VAR DIFF = NOW()-'Fact Readings (dataflow)'[Last Communication (meter)]

VAR NumOfMinutes = DIFF * 24

    * 60
    
VAR DAYS =

    IF ( DIFF >= 1, INT ( DIFF ), BLANK () )
    
VAR HOURS =

    INT ( ( DIFF - DAYS ) * 24 )
    
VAR MINUTES = NumOfMinutes

    - ( DAYS * 24
    
    * 60 )
    
    - ( HOURS * 60 ) 
    
//TEXTS
VAR DaysText =

    IF ( DAYS > 1, FORMAT ( DAYS, "00" ) & "days", IF ( Days = 1, FORMAT ( DAYS, "00" ) & "day"))
    
VAR HoursText =

    FORMAT ( HOURS, "00" ) & " hrs "
    
VAR MinutesText =

    FORMAT ( MINUTES, "00" ) & "  mins ago"
    
RETURN

    IF (
    
        NOT ( ISBLANK ( [Last Communication (meter)] ) ),
        
        COMBINEVALUES ( " ", DaysText, HoursText, MinutesText )
        
    )

# Last Meter Reading (kW) =
CALCULATE(SUM('Fact Readings (dataflow)'[Consumption (kWh)]), 'Fact Readings (dataflow)'[NewDateTime]= MAX('Fact Readings (dataflow)'[NewDateTime]))


# Title local date and time = 
"Local Date and Time: " & [Today's Day Name] & " "&NOW()

# Total Last Calendar Week = 
CALCULATE (SUM ( 'Fact Readings (dataflow)'[Consumption (kWh)] ), DimDates[Week Offset] = -1)


# Total Same Calendar Month LY = 
CALCULATE (SUM ( 'Fact Readings (dataflow)'[Consumption (kWh)] ), DimDates[Month Offset] = -12)


# Total Same Day Last Week = 
CALCULATE(
    SUM('Fact Readings (dataflow)'[Consumption (kWh)]), DimDates[Date]=TODAY()-7)

# Total Today = 
CALCULATE(
    SUM('Fact Readings (dataflow)'[Consumption (kWh)]), DimDates[Date]=TODAY())

# Total Yesterday = 
CALCULATE(
    SUM('Fact Readings (dataflow)'[Consumption (kWh)]), DimDates[Date]=TODAY()-1)

# Total week before full last week = 
CALCULATE (SUM ( 'Fact Readings (dataflow)'[Consumption (kWh)]), DATESINPERIOD(DimDates[Date], TODAY()-7, -7, day))

# Latest Weather Condition = 
LASTNONBLANKVALUE (FactWeather[Date_Time_SQL_Conversion], MAX( ( FactWeather[WeatherDescription] ) ))
