
One of the most basic steps in LiDAR analysis in forest resources is assigning the raw points in the point cloud to a grid. This allows us to simplify many problems to a space in which we are thinking about raster cells instead of three  diemensional points. It is computationally advantageous as well.

We will be using four packages for this process, `laspy` to read in our .las file, `numpy` for matrix computations, `pandas` for data frames, and `matplotlib` to see our product.


```python
# Import our packages
import numpy as np
import pandas as pd
import laspy
import matplotlib.pyplot as plt
```

We will call our gridding function `grid` and it will take two arguments, our `las` file object and some grid size `c`.


```python
# Read in the las data
las1 = laspy.file.File("data/sample.las")

def grid(las, c):
    # Determine the number of rows (m) and columns (n)
    # Some python interpreters complain about the np float datatype
    # So we will convert to integers just in case
    m = int(np.floor((max(las.y) - min(las.y)) / c) + 1)
    n = int(np.floor((max(las.x) - min(las.x)) / c) + 1)
    
    # Create bins
    bins_x = np.digitize(las.x, np.linspace(min(las.x), max(las.x), n + 1))
    bins_y = np.digitize(las.y, np.linspace(min(las.y), max(las.y), m + 1))

    # Add bins and las data to a new dataframe
    df = pd.DataFrame({'x': las.x, 'y': las.y, 'z': las.z, 'bins_x': bins_x, 'bins_y': bins_y})
    return(df)
```

Our grid function is complete, but lets dissect what is going on in detail a bit more, specifically the computations in `bins_x` and `bins_y`.

`np.digitize` requires to arguments. The first is a vector of numbers, in our case it is a vector of a LiDAR dimension. The second is a vector of bin "edges". We construct this using `np.linspace` which generates a sequence of numbers from the first argument to the second, the third argument dictates how many numbers are generated. The spacing between these numbers is even. Below is a simple example of `np.linspace` where we generate 5 numbers (the third argument) evenly spaced between 2 and 7.


```python
np.linspace(2, 7, 5)
```




    array([ 2.  ,  3.25,  4.5 ,  5.75,  7.  ])



We now have our bin edges, and via `np.digitize` we now have a list of integers in both `bins_x` and `bins_y` that correspond to their respective grid cells. The last step is to merge them using a call to `pd.DataFrame`. Below is an example output of `grid` for our `las1` file and a 2 meter pixel resolution.


```python
gridded_df1 = grid(las1, 2)
gridded_df1.head(15)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>bins_x</th>
      <th>bins_y</th>
      <th>x</th>
      <th>y</th>
      <th>z</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>470094.87</td>
      <td>5016466.27</td>
      <td>521.20</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1</td>
      <td>470094.93</td>
      <td>5016465.58</td>
      <td>521.78</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>1</td>
      <td>470095.31</td>
      <td>5016465.31</td>
      <td>521.83</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>1</td>
      <td>470095.27</td>
      <td>5016465.97</td>
      <td>521.83</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>1</td>
      <td>470095.12</td>
      <td>5016466.78</td>
      <td>518.73</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1</td>
      <td>2</td>
      <td>470094.91</td>
      <td>5016467.00</td>
      <td>513.54</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1</td>
      <td>2</td>
      <td>470095.11</td>
      <td>5016467.42</td>
      <td>519.25</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1</td>
      <td>2</td>
      <td>470094.90</td>
      <td>5016467.64</td>
      <td>514.29</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1</td>
      <td>3</td>
      <td>470094.95</td>
      <td>5016468.85</td>
      <td>516.87</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1</td>
      <td>3</td>
      <td>470094.91</td>
      <td>5016469.53</td>
      <td>516.60</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1</td>
      <td>4</td>
      <td>470094.89</td>
      <td>5016472.72</td>
      <td>513.22</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1</td>
      <td>4</td>
      <td>470094.97</td>
      <td>5016472.00</td>
      <td>514.10</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1</td>
      <td>4</td>
      <td>470094.96</td>
      <td>5016471.39</td>
      <td>513.10</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1</td>
      <td>3</td>
      <td>470095.00</td>
      <td>5016470.70</td>
      <td>513.37</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1</td>
      <td>3</td>
      <td>470095.10</td>
      <td>5016469.97</td>
      <td>515.12</td>
    </tr>
  </tbody>
</table>
</div>



Every point is now sorted into its respective grid cell. This has huge advantages for us moving forward, as we can now summarize and extract data for a given grid cell in the point cloud. One of the most straightforward applications of this is the generation of a canopy height model. A canopy height model describes the elevation of the canopy in each pixel, usually in a raster format.

For our purposes we will stick to a simple plot to view our data. We will make a function that takes the `gridded_df1` from our last function's output and presents it in a nice plot.


```python
def matrix_plot(gridded_df):
    # Group by the x and y grid cells
    group_df = gridded_df[['bins_x', 'bins_y', 'z']].groupby(['bins_x', 'bins_y'])
    
    # Summarize (i.e. aggregate) on the max z value and reshape the dataframe into a 2d matrix
    plot_mat = group_df.agg({'z': 'max'}).reset_index().pivot('bins_y', 'bins_x', 'z')
    
    # Plot the matrix, and invert the y axis to orient the 'image' appropriately
    plt.matshow(plot_mat)
    plt.gca().invert_yaxis()
    
    # Show the matrix image
    plt.show()
```

The powerful part here is being able to leverage `pandas` `groupby` function, which, as the name suggests, groups the data frame by one or more columns and summarize the data within those columns. In this specific scenario we wish to summarize on the maximum value of z in each group (i.e. each cell). Once the data frame is grouped, then we aggregate using the `agg` method.

We can now see the fruits of our labor in the plot below.


```python
matrix_plot(gridded_df1)
```


![png](Canopy_Height_Model_files/Canopy_Height_Model_11_0.png)


Thanks to the `grid` function, we can easily resize our pixels if we desire something more fine-grained:


```python
matrix_plot(grid(las1, 0.5))
```


![png](Canopy_Height_Model_files/Canopy_Height_Model_13_0.png)


There is, of course, some fine tuning to do yet. For example, in our first plot the edges seem to have NaN values. Also, our axes should reflect the X Y coordinates in UTM space, but instead refer to the bin ids. I will leave these fixes as an exercise to the reader.
