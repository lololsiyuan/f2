# Scale

## Definition of Scale

Scale is the conversion bridge from data to graph and is responsible for the conversion of the original data to the values of interval [0, 1] (the conversion from the raw data to [0, 1] interval is called the normalization).

Different data types correspond to different scales, for example:

1. Continuous data type. For example, `0, 1, 2, ..., 9, 10` can be turned into `0.1, 0,2, 0.3, ..., 0.9, 1` through scale, and using invert, those values can be restored to the original values.
2. Classified data type. For example, `['male', 'female']` can be turned into [0, 1] using scale, and then it can also be restored to the original value through inverting.

## Function of Scale

Scales are used in graphical syntax to accomplish the following functions:

1. Convert data to the range [0, 1] to facilitate the mapping of raw data to graphics, such as position, color, and size;
2. Invert the normalized data back to the original value. For example, if the `category a` is converted to 0.2, the corresponding `0.2` needs to be inverted back to `category a`;
3. Dividing Data, used in axis, ranges of values in lengends, and category information.

## Type Of Scale

The type of scale is determined by the type of the original data, so you need to understand how F2 categorizes the data before we talk about types of scales.

We divide the data into two types, according to wether the values of data are continuous:

1. Classified (non-continuous) data, can be further divided into ordered classified data and unordered category data;
2. Continuous data (time is continuous)

Example:

```js
const data = [
  { month: 'January', temperature: 7, city: 'Tokyo' },
  { month: 'February', temperature: 6.9, city: 'New York' },
  { month: 'March', temperature: 9.5, city: 'Tokyo' },
  { month: 'April', temperature: 14.5, city: 'Tokyo' },
  { month: 'May', temperature: 18.2, city: 'Berlin' }
];

// config scale
chart.scale({
  month: {
    alias: 'Month' // define property name
  }, 
  temperature: {
    alias: 'Temperature' // define property name
  }
});
```

In the above, `month` and `city` are the classfied data, but the difference is that `month` is ordered while `city` is unordered and `temperature` here is continuous.

According to the above classification method, F2 provides the following different scale types:

| data type  | scale type   |
| ---------- | ------------ |
| Continuous | Linear       |
| Classified | Cat, TimeCat |

In addtion, we provide a scale called `Identity`, it is used for operations of **constant** in data source.

For all scales generated by F2, you have the following attributes, which can be configured by the developers.

```js
{
  type: {String}, // type of scale
  range: {Array}, // range of scale conversions, defaults to [0, 1]
  alias: {String}, // alias for data attributes for persinalized display of legends, tooltip and axes
  ticks: {Array}, // stores the scale information on the axis
  tickCount: {Number}, // number of scales on the coordinate axis, different scale types correspond to different default values
  formatter: {Function} // callback, used to format the display of the scales of axies, affects data displayed on the axis, lengend and tooltip
}
```

The mechanism by which F2 generates scales by default is as follows:

* See if user has defined the data type of the corresponding field, see [Column Definitions](https://antv.alipay.com/zh-cn/g2/3.x/tutorial/how-to-scale.html)
* If user hasn't defined the data type, use the field of first data to deduce scale type
  * If there is no corresponding field, scale is going to be defined as `Identity`
  * If the field is a number, scale is going to be defined as `Linear`
  * if the field is a string, scale is going to be defined as `Cat`
  * User need to use Column Definition to manually specify a `TimeCat`

Let's take a closer look at the types of scales below:

### Linear

Consecutive data values, such as [1, 2, 3, 4, 5], in addition to common attributes, it also include the following extra attributes:

```js
{
  nice: {Boolean}, // The default value is true, it is used to optimize the range of values and evenly distribute the scale on the axes. For example, if nice is true, [3, 97] will be adjusted to [0, 100] for optimization.
  min: {Number}, // minimum value of the range
  max: {Number}, // maximum value of the range
  tickCount: {Number}, // define the scale of axis
  tickInterval: {Number}, // Used to specify the distance between the scales of the coordinate axis, it is the difference of the original data values. Note that tickCount and tickInterval cannot be specified at the same time
}
```

For example, 

```js
const data = [
  { name: '张三', score: 53 },
  { name: '王五', score: 92 }
];

chart.source(data);
chart.point().position('name*score').color('name');
chart.render();
```

<img src="https://gw.alipayobjects.com/zos/rmsportal/ldVMrTNHMHJbPaukJhFe.png" style="width:400px">

**Comments**

*The default range of scales for scores is 50-95, which is the effect of `nice: true`, it makes it look clearer.*

Student scores' range is actually 0-100, so 50-90 does not meet our needs, we can then limit the range using `min` and `max`:

```js
const data = [
  { name: '张三', score: 53 },
  { name: '王五', score: 92 }
];

chart.source(data, {
  score: {
    min: 0,
    max: 100
  }
});

chart.point().position('name*score').color('name');
chart.render();
```

<img src="https://gw.alipayobjects.com/zos/rmsportal/mQVOhgkaViFkGojDREHR.png" style="width:400px">

### Cat

Classified scale type. In addition to having common attrtibutes, developers can also set the `values` aatributes:

```js
{
  values: {Array} // specify the category value of current field
}
```

When F2 generates a `Cat` scale, the value of `values` attribute is generally obtained directly from the original data, but for the following two scenarios, the user is required to manually specify the `values`:

1. When you need to specify the sorting order, for example: the original value of the type field is ['max', 'min', 'medium'], and we wanr to specify that order to ['min' 'medium', 'max'] on axis or legend. In order to accomplish this, `Cat` needs to be configured as follows:

   ```js
   const data  = [
     { a: 'a1', b: 'b1', type: 'min' },
     { a: 'a2', b: 'b2', type: 'max' },
     { a: 'a3', b: 'b3', type: 'medium' }
   ];
   chart.scale('type', {
     type: 'cat',
     values: [ 'min', 'medium', 'max' ]
   });
   ```

   If you don't specify the `values` field, the default order is: 'min', 'max', 'medum'.

2. If the classified type in the data is represented by an enumeration, you also need to specify `values`:

   ```js
   const data  = [
     { a: 'a1', b: 'b1', type: 0 },
     { a: 'a2', b: 'b2', type: 2 },
     { a: 'a3', b: 'b3', type: 1 }
   ]
   chart.scale('type', {
     type: 'cat',
     values: [ 'min', 'medium', 'max' ]
   });
   ```

   The `Cat` type must be specified here, and the value of `values` field must correspond to the enumeration type by index.

### TimeCat

`TimeCat` scale corresponds to time data, which by default is sorted.

`TimeCat` is a subclass of the `Cat` scale, and has its own properties in addition to common properties and properties belong to `Cat` scale:

```js
{
  mask: {String}, // specify the display format of time data, defaults to 'YYYY-MM-DD'
}
```


