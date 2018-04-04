
An integral part of LiDAR data processing is computing an underlying digital terrain model (DTM) so that we can easily compute the heights of objects in our point cloud. There are a multitude of ways to do this, and this chapter will go over one of the most basic approaches as a way to apply the concepts covered in the previous exercise. More sophisticated algorithms should be used for operational LiDAR analysis, this is only a simple example.

The basic idea is to find the minimum point within each grid cell of a gridded point cloud. We will classify this point as "ground point" and rewrite to a new LAS file. We can take advantage of some functions we developed in the "Canopy Height Model" exercise to do this.

Below we import our functions from the "Canopy Height Model" exercise and grid the input las file.


```python
import laspy
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Read in the las data
las1 = laspy.file.File("data/sample.las")

def grid(las, c):
    # Determine the number of rows (m) and columns (n)
    # Some python interpreters complain about the np float datatype
    # So we will convert to integers just in case
    m = int(np.floor((max(las.y) - min(las.y)) / c) + 1)
    n = int(np.floor((max(las.x) - min(las.x)) / c) + 1)
    
    # Create bins
    bins_x = np.digitize(las.x, np.linspace(min(las.x), max(las.x), n))
    bins_y = np.digitize(las.y, np.linspace(min(las.y), max(las.y), m))

    # Add bins and las data to a new dataframe
    df = pd.DataFrame({'x': las.x, 'y': las.y, 'z': las.z, 'bins_x': bins_x, 'bins_y': bins_y})
    return(df)

gridded_las = grid(las1, 3)
gridded_las.head(5)
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
  </tbody>
</table>
</div>



To preserve some efficiency, the information we want to retrieve is a list of indices that correspond to the points in the original point cloud with the minimum z value in each cell. We can do this using a boolean mask and the `pandas` `transform` function.


```python
# Group a subset of the data by bin ID
gridded_las_group = gridded_las[["bins_x", "bins_y", "z"]].groupby(["bins_x", "bins_y"])

# Transform the groups and find the indices of the minimum z for each group (i.e. each cell)
ground_mask = gridded_las_group['z'].transform(min) == las1.z
```

Now that we have the mask available, we can use it to write a new las file and a host of other operations. Because `numpy` and `pandas` play well with eachother, we can directly use the `pandas` derived mask as a mask for the `las1.x` `numpy` array.


```python
# Get the ground points
las1.x[ground_mask]
las1.y[ground_mask]
las1.z[ground_mask]

# Write to file

outlas = laspy.file.File("./data/simple_Ground.las", mode = "w",
                header = las1.header)

outlas.x = las1.x[ground_mask]
outlas.y = las1.y[ground_mask]
outlas.z = las1.z[ground_mask]

outlas.close()
```

There are still some issues, most notably at the edges of the point cloud that should be addressed. More sophisticated ground filters are ideal for operational use, however. One solution is to consider only the "inner" portion of the tile. This would remove converns at the edge.
