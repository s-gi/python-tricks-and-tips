## Understanding How Matplotlib Represents Event Data
In Matplotlib, `event.xdata` (and similar attributes like `event.ydata`) represents the data coordinates of a mouse click or interaction, and these are returned in the format that corresponds to the data type of the plot's axes.

If `event.xdata` contains data in a "Matplotlib format," it's typically referring to the following concepts:

### 1. **Data Representation for Different Axis Types**
Matplotlib plots can use different kinds of data for the x-axis (and y-axis) based on the type of plot and data used:
- **Numerical data**: If your x-axis is numerical, `event.xdata` will return the x-value in its numerical form (like a float).
- **Datetime objects**: If your x-axis is based on `datetime` or `pandas.Timestamp` objects, `event.xdata` will return the corresponding numerical representation (often in days, seconds, or some other time unit).
- **Categorical data**: If the x-axis uses categorical data (such as strings or labels), `event.xdata` will return the index of the category that was clicked.

### 2. **The Format for `event.xdata`**
The format of `event.xdata` corresponds to the axis type used by the plot. Here’s how it works:

#### 2.1 **Numerical Axes (e.g., Float or Integer Data)**
If the x-axis represents numerical values, `event.xdata` will directly give you the corresponding float or integer value that was clicked.

#### 2.2 **Datetime Axes**
If the x-axis uses dates or times, Matplotlib internally stores and handles dates as a numerical representation, typically a floating-point number representing the number of days since the Unix epoch (1970-01-01 00:00:00) or a similar reference time.

For instance:
- If your x-axis represents timestamps or dates (like `pd.Timestamp` or `datetime`), `event.xdata` will give you a floating-point number representing the number of days (or seconds, depending on the plot) since the epoch. 
- When you click on the plot, `event.xdata` may look like a large float number (such as `20096.66871327352`), which corresponds to the date and time of the click.

#### 2.3 **Categorical Axes**
If the x-axis uses categorical data (strings or labels), Matplotlib will map these categories to indices (i.e., numeric representations). For example:
- If you have categories like `['A', 'B', 'C']`, clicking on "B" might return the value `1`, as it's the index of "B" in the list.
- This is why `event.xdata` can sometimes return integer values corresponding to the category index, rather than the raw category label.

### 3. **Why It Might Appear in "Matplotlib Format"**
When you see "Matplotlib format," it generally means that the data has been transformed into a format that Matplotlib can plot efficiently. This could mean:
- **Dates are represented as floats**: Dates and times are internally represented as floats or integers (e.g., days since epoch, seconds since epoch) to allow for efficient plotting and scaling.
- **Data is transformed for scaling or normalization**: If your data is normalized or scaled (e.g., `event.xdata` represents a percentage of the total range), it might appear as a scaled value.
- **Categorical data is indexed**: Categorical data might appear as indices instead of raw labels.

### Example: `event.xdata` with Datetime Plot

Consider the following example:

```python
import matplotlib.pyplot as plt
import pandas as pd

# Create a simple time series
dates = pd.date_range('2025-01-01', periods=5, freq='D')
values = [1, 3, 5, 7, 9]

fig, ax = plt.subplots()
ax.plot(dates, values)

# Event handling function
def on_click(event):
    print(f"Clicked x-data: {event.xdata}")

# Connect the event handler
fig.canvas.mpl_connect('button_press_event', on_click)

plt.show()
```

In this example:
- The x-axis represents dates (`datetime` objects).
- When you click on the plot, `event.xdata` will give you a float representing the number of days since the Unix epoch.

If you print `event.xdata`, you might see something like this:

```
Clicked x-data: 20096.66871327352
```

This value (`20096.66871327352`) represents the date of the click as a floating-point number, corresponding to the number of days since `1970-01-01 00:00:00`. Matplotlib uses this internal float representation for dates for efficiency in plotting and computations.

### 4. **Converting Back to Original Format**
If you want to convert the `event.xdata` value (e.g., a float) back to a human-readable timestamp or datetime, you can do the following:

```python
# Convert the float (days since epoch) back to a datetime
clicked_timestamp = pd.to_datetime(event.xdata, unit='D', origin='unix')
print(clicked_timestamp)
```

This converts the float value back into a `pandas.Timestamp` or Python `datetime`.
> [!TIP]
> - The conversion can be also carried out as in the snippet below as suggested in [this](https://stackoverflow.com/questions/34725701/how-to-get-data-in-datetime-format-from-plot-with-events-in-python/34726450#34726450) stackoverflow entry.
> ```python
> import matplotlib.dates as mdates
> # Convert the float (days since epoch) back to a datetime with mdates.num2date
> clicked_timestamp = mdates.num2date(event.xdata)
> print(clicked_timestamp)
> ```
> - If instead you want to convert human-readable timestamp or datetime to `event.xdata` you might use python's matplotlib.dates.date2num, converting numpy array to matplotlib datetimes. See [this](https://stackoverflow.com/questions/34843513/python-matplotlib-dates-date2num-converting-numpy-array-to-matplotlib-datetimes) stackoverflow entry on the topic.

### 4.1 Code snippet: convert datetime to matplotlib format
```python
import pandas as pd
import matplotlib.dates as dates
import numpy as np

t = pd.date_range(pd.Timestamp.today(), periods=5)
print(f'Original date range: {t}')

# Convert t to a NumPy array
t_np = t.to_numpy()

# This line below did not raise an error, as in the original post. I don't know why.
plt_dates1 = dates.date2num(t_np)
print(f'Conversion to Matplotlib format with date2num: {plt_dates1}')

# The workaround below works not just as an alternative,
# but also gives more insight into the Matplotlib format.
z = np.array([0]).astype(t_np.dtype)
plt_dates2 = (t_np - z) / np.timedelta64(1, 'D')
print(f'Conversion to Matplotlib format with workaround (t_np - z) / np.timedelta64: {plt_dates2}')

# Assert that the two conversions are equal
assert np.allclose(plt_dates1, plt_dates2), "The two date conversions are not equal"
```
### When pandas.Timeseries have a perticular frequency: use pandas.to_datetime
In [this](https://stackoverflow.com/questions/54035067/matplotlib-event-xdata-out-of-timeries-range/54036315#54036315) the very related entry in stackoverflow, `matplotlib.dates.num2date` do not return the correct value as in

```python
import numpy as np
import pandas as pd

# uncomment if default backend do not support interactive features
# import matplotlib
# matplotlib.use('TkAgg')  # Set the backend to TkAgg before importing pyplot

import matplotlib.pyplot as plt
import matplotlib.dates as mdates

def on_click(event):
    if event.inaxes is not None:
        # provide raw and converted x data
        print(f"{event.xdata} --> {mdates.num2date(event.xdata)}")
        # add a vertical line at clicked location
        line = ax.axvline(x=event.xdata)
        # Convert the float (days since epoch) back to a datetime
        clicked_timestamp = pd.to_datetime(event.xdata, unit=ax.lines[0].get_xdata()[0].freqstr, origin='unix')
        print(f"pandas.to_datetime --> {clicked_timestamp}")
        plt.draw()

t = pd.date_range('2015-11-01', '2016-01-06', freq='h')
y = np.random.normal(0, 1, t.size).cumsum()

df = pd.DataFrame({'Y':y}, index=t)

fig, ax = plt.subplots()
line = None
df.plot(ax=ax)
fig.canvas.mpl_connect('button_press_event', on_click)
plt.show()
```
Note that in this case `matplotlib.dates.num2date` fails whereas `pandas.to_datetime`, with the appropriate frequency, in this case 'h', is still working. See output:

```
402204.54193548387 --> 3071-03-14 13:00:23.225800+00:00
pandas.to_datetime --> 2015-11-19 12:32:30.967741921
```
Note tha also pandas.Period will work, as far as the correct frequency is provided

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt


def on_click(event):
    print(pd.Period(ordinal=int(event.xdata), freq='h'))


t = pd.date_range('2015-11-01', '2016-01-06', freq='h')
y = np.random.normal(0, 1, t.size).cumsum()

df = pd.DataFrame({'Y': y}, index=t)

fig, ax = plt.subplots()
df.plot(ax=ax)
fig.canvas.mpl_connect('button_press_event', on_click)
plt.show()
```

In thiscase, however, the output is trucated to the nearest hour. See below an example of output.

```
2015-11-08 11:00
pandas.to_datetime --> 2015-11-08 11:54:11.612903130
```

### Conclusion
`event.xdata` contains data in a "Matplotlib format" because Matplotlib internally handles the data in a numerical form that is optimized for plotting. For date/time plots, this often means that dates are represented as floating-point numbers (e.g., days or seconds since the Unix epoch). The format you see in `event.xdata` depends on the type of data used in your plot's x-axis (numerical, datetime, or categorical).

## Enabling Interactive Features
When the backend used by default do not support interactive features, e.g sometime in PyCharm, you need to use one that Matplotlib uses to render plots and handle interactions, like e.g. `matplotlib.use('TkAgg')` 

### Understanding Matplotlib Backends

Matplotlib supports several **backends**, each designed to work with different environments, GUI toolkits, and rendering methods. A **backend** is responsible for how Matplotlib handles plotting, rendering, and user interaction (like clicks, zoom, and panning).

#### Key Matplotlib Backends:

1. **TkAgg**:
   - This backend uses **Tkinter**, which is a standard GUI library in Python, to render and interact with plots.
   - It's widely used and supports interactive features, like handling mouse clicks, zooming, and panning.
   
2. **Agg**:
   - This is a **non-interactive backend** designed for generating plots without user interaction. It is primarily used for saving plots to files (like PNG, SVG, etc.).
   - This backend does not support mouse events because it is not designed for interactive plotting.

3. **Qt5Agg**:
   - Uses the **Qt** framework (like PyQt or PySide) for rendering interactive plots.
   - Also supports interaction, including mouse events.

4. **WebAgg**:
   - A backend that works in the browser and can display interactive plots in a web application.
   
5. **MacOSX**:
   - A backend used on macOS, relying on macOS's native graphics system.
   - Supports interactive features, but may require additional setup for certain systems.

### Why `matplotlib.use('TkAgg')` is Required in PyCharm

In PyCharm, the default backend for Matplotlib may not always support interactive features (like mouse click events), especially when you're running scripts in non-interactive environments (such as terminal-based plots). In these cases, PyCharm's default backend might be **Agg** or another non-interactive backend, which does not handle GUI-based user input (like mouse clicks).

To enable interaction in PyCharm, you explicitly set the backend to **TkAgg**, which uses the Tkinter GUI framework. This backend ensures that the plot window can handle mouse events such as clicks, and it also enables zooming and panning.

### How `matplotlib.use('TkAgg')` Works

By calling `matplotlib.use('TkAgg')` before importing `matplotlib.pyplot`, you are telling Matplotlib to use the **TkAgg** backend, which enables interactive features:

```python
import matplotlib
matplotlib.use('TkAgg')  # Set the backend to TkAgg before importing pyplot

import matplotlib.pyplot as plt
```

- **TkAgg** backend ensures that the plot window uses the Tkinter event loop for interaction.
- This backend allows the plotting window to remain open and respond to events like mouse clicks.

### Complete Example with `TkAgg`:

```python
import matplotlib
matplotlib.use('TkAgg')  # Set the backend to TkAgg

import matplotlib.pyplot as plt
import pandas as pd

# Create a simple time series
dates = pd.date_range('2025-01-01', periods=5, freq='D')
values = [1, 3, 5, 7, 9]

# Create a plot with the data
fig, ax = plt.subplots()
ax.plot(dates, values)

# Event handling function to capture mouse click events
def on_click(event):
    print(f"Clicked x-data: {event.xdata}")

# Connect the event handler to the 'button_press_event'
fig.canvas.mpl_connect('button_press_event', on_click)

# Display the plot
plt.show()
```

### Steps to Run the Code:

1. **Install Required Libraries**: If you haven't already, make sure `matplotlib` and `pandas` are installed:

   ```bash
   pip install matplotlib pandas
   ```

2. **Create a New Python File in PyCharm**: Open PyCharm and create a new Python file (e.g., `interactive_plot.py`), and paste the above code into it.

3. **Run the Script**: Click the **Run** button (green triangle icon) or press `Shift + F10` to run the script. This will open a new window displaying the plot.

4. **Click on the Plot**: When the plot window opens, click on various points on the plot, and you'll see the `event.xdata` printed in the console. The value represents the number of days since the Unix epoch for the clicked point.

### Expected Output in the Console:
For example, clicking on different x-values (dates) will output something like:

```
Clicked x-data: 20096.5
Clicked x-data: 20097.5
Clicked x-data: 20098.5
```

Here, `event.xdata` is a floating-point value representing the number of days since the Unix epoch (`1970-01-01`).

### When You Don't Need `matplotlib.use('TkAgg')`
If you're running your script in an environment that already supports interactive plotting (like Jupyter notebooks or some Python IDEs), Matplotlib might automatically use an appropriate backend (like **TkAgg** or **Qt5Agg**) that supports interactivity without needing to set it explicitly. In these cases, you don't need to call `matplotlib.use('TkAgg')`.

### What Happens Without `matplotlib.use('TkAgg')`?
If you don't specify a backend in PyCharm, and Matplotlib defaults to a non-interactive backend like **Agg**, the plot will still render, but:
- It won't support interactive features like mouse clicks.
- The plot will close immediately after rendering because the script finishes execution without waiting for interaction.

### Summary:

- **PyCharm** may default to a non-interactive backend like **Agg**, which doesn't support mouse events.
- By setting `matplotlib.use('TkAgg')`, you're explicitly telling Matplotlib to use the **TkAgg** backend, which supports interactive features like click events.
- This is needed in PyCharm or other IDEs when you're running scripts outside of environments like Jupyter notebooks that automatically handle interactive plotting.
