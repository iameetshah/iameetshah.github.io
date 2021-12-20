---
title: Stacked Bar Chart with Corner Radius Renderer
author: Meet Shah
date: 2021-12-20
category: Jekyll
layout: post
---

In this article we will look into creating a corner-radius renderer for stacked bar chart.
(Refer `Final Result` for screenshot after using corner-radius renderer). Assuming you have already installed 
[Charts Library](https://github.com/danielgindi/Charts) by adding to Podfile of your project and imported Charts to required UIViewController.


**Final Result :**

![screenshot5](../../../../assets/Vertical_Stacked_Chart.png){: width="280" }


Let’s start by creating required months and associated values. 

```swift
    let months = ["J","F","M","A","M","J","J","A","S","O","N","D"]
    let percentages = [52.0, 40.0, 68.0, 100.0, 0.0, 98.0, 75.0, 100.0, 20.0, 10.0, 0.0, 99.0]
```

Next we need to create and connect IBOutlet of `barChartView`. 
Now, create `setupVerticalChart()` method to define required graph Configurations/Settings (they are self-explanatory):

```swift
    func setupVerticalChart() {
    // Graph Position
    barChartView.extraLeftOffset = 0
    barChartView.extraTopOffset = 0
    barChartView.extraBottomOffset = 0
    barChartView.extraRightOffset = 0
    
    let xAxis = barChartView.xAxis
    let rightAxis = barChartView.rightAxis
    barChartView.leftAxis.enabled = false
    xAxis.labelPosition = .bottom
    barChartView.legend.enabled = false
    barChartView.drawGridBackgroundEnabled = false
    xAxis.granularity = 1.0
    xAxis.labelCount = 12
    
    barChartView.pinchZoomEnabled = false
    barChartView.doubleTapToZoomEnabled = false
    
    barChartView.backgroundColor = .white
    
    // Graph X Axis and Right Axis Color
    xAxis.axisLineColor = .clear//UIColor.black.withAlphaComponent(0.2)
    xAxis.gridColor = .clear//UIColor.black.withAlphaComponent(0.2)
    xAxis.labelTextColor = UIColor.black.withAlphaComponent(0.5)
    rightAxis.gridColor = .clear//UIColor.black.withAlphaComponent(0.2)
    rightAxis.axisLineColor = .clear//UIColor.black.withAlphaComponent(0.2)
    rightAxis.labelTextColor = UIColor.black.withAlphaComponent(0.5)
    
    // Graph X Axis and Right Axis Font
    rightAxis.labelFont = UIFont.systemFont(ofSize: 12)
    xAxis.labelFont = UIFont.systemFont(ofSize: 12)
    
    rightAxis.drawZeroLineEnabled = false
    rightAxis.drawAxisLineEnabled = false
    barChartView.delegate = self
  }
```

We need to create 2 helper method `dataSetWith()` to create `BarChartDataSet` and `setupRightAxisFormatter` for formatting values with "%" suffix:

```swift
    func setupRightAxisFormatter() {
        let rightAxisFormatter = NumberFormatter()
        rightAxisFormatter.positiveSuffix = "%"
        let rightAxis = barChartView.rightAxis
        rightAxis.valueFormatter = DefaultAxisValueFormatter(formatter: rightAxisFormatter)
        rightAxis.axisMinimum = 0
        rightAxis.axisMaximum = 100
        rightAxis.granularity = 25
    }
    func dataSetWith(entries: [BarChartDataEntry],
                     colors: [UIColor] = [.black],
                     highlightColor: UIColor,
                     label: String = "") -> BarChartDataSet {
        
        let barDataSet = BarChartDataSet(entries: entries, label: label)
        barDataSet.drawIconsEnabled = false
        barDataSet.drawValuesEnabled = false
        barDataSet.colors = colors
        barDataSet.highlightColor = highlightColor
        barDataSet.highlightAlpha = 1.0
        barDataSet.highlightLineWidth = 0
        
        return barDataSet
    }
```

Now we are ready to update Chart data for 12 months with respective values of each month. 
Since we are working with Stacked bar chart, we need to calculate 2 values i.e. `val1` & `val2` 
wherein `val1` is value depicted by orange color & `val2` with light gray color.

After creating `BarChartData` from `BarChartDataSet` we notify ChartView of dataSet modification with `barChartView.notifyDataSetChanged()`:

```swift
       func setupData() {
        var barEntries = [BarChartDataEntry]()
      
        for interval in 0..<months.count {
            let val1 = Double(percentages[interval])
            let val2 = 100 - val1
            barEntries.append(BarChartDataEntry(x:  Double(interval), yValues: [val1, val2]))
        }
        
        let barDataSet = dataSetWith(entries: barEntries,
                                     colors: [UIColor.orange.withAlphaComponent(0.7), UIColor.black.withAlphaComponent(0.1)],
                                     highlightColor: UIColor.orange.withAlphaComponent(1.0),
                                             label: "label")

        barChartView.xAxis.valueFormatter = IndexAxisValueFormatter(values: months)
        setupRightAxisFormatter()

        let barData = BarChartData(dataSet: barDataSet)
        barData.barWidth = 0.65
        
        barDataSet.barBorderWidth = 0.5
        barDataSet.barBorderColor = UIColor.black.withAlphaComponent(0.1)

        
        barChartView.data = barData
        barChartView.fitBars = true
        barDataSet.axisDependency = .right

        barChartView.notifyDataSetChanged()
    }
```
Finally, Update `viewDidLoad()` method  by adding `setupVerticalChart()` & `setupData()` 
so that on intialisation itself Bar chart is first configured with required settings & then loaded with data :

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        setupVerticalChart()
        setupData()
    }
```

Inorder to create `CornerRadiusStackBarRenderer` we need to first jump to the definition of `BarChartRenderer` 
for `initBuffers`, `prepareBuffer`, `drawDataSet` & `drawHighlighted` methods. 
Next create a new class of `CornerRadiusStackBarRenderer` which inherits from `BarChartRenderer` so we can override the behaviour as per our needs.
Create `setupCorner()` method and update `drawDataSet`, `drawHighlighted` methods to set up corner-radius : - 

```swift
    func setupCorner(context:CGContext, dataSet: IBarChartDataSet, index: Int, stackIndex:Int, barRect:CGRect) {
    if let currentEntry = dataSet.entryForIndex(index) as? BarChartDataEntry {
        let bezierPath = UIBezierPath(roundedRect: barRect, byRoundingCorners: [.allCorners], cornerRadii: CGSize(width: cornerRadius, height: cornerRadius))
        if currentEntry.yValues![stackIndex] == 0 || currentEntry.yValues![stackIndex] == 100 {
            let roundedPath = bezierPath.cgPath
            context.addPath(roundedPath)
            context.fillPath()
        }
        else {
            if stackIndex == 0 {
                let bezierPath = UIBezierPath(roundedRect: barRect, byRoundingCorners: [.bottomLeft, .bottomRight], cornerRadii: CGSize(width: cornerRadius, height: cornerRadius))
                let roundedPath = bezierPath.cgPath
                context.addPath(roundedPath)
                context.fillPath()
            }
            else {
                let bezierPath = UIBezierPath(roundedRect: barRect, byRoundingCorners: [.topLeft, .topRight], cornerRadii: CGSize(width: cornerRadius, height: cornerRadius))
                let roundedPath = bezierPath.cgPath
                context.addPath(roundedPath)
                context.fillPath()
            }
        }
    }
  }
```

Update `setupVerticalChart()` method inorder to start using the corner-radius renderer.

```swift
  barChartView.renderer = CornerRadiusStackBarRenderer(dataProvider: barChartView,
                                                         animator: barChartView.chartAnimator,
                                                         viewPortHandler: barChartView.viewPortHandler)
```


Hope above post was informative and useful. You can check the source code on github : [HERE](https://github.com/iameetshah/verticalBarChartDemo)
