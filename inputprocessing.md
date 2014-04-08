# Input processing

The following explains what each input processor does and details what the argument requirements are.

Input processes are executed sequentially with the result being passed back for further processing by the next processor in the input processing list. This makes it possible to say: take an input called current multiply it with another input called voltage, log it to a feed called power, convert the power variable to kWh/d data and log that to a feed called kWhddata.

Each of these processes uses the value returned by the previous process in the chain, if it is the first in the chain it will use the raw input value. So the input value and therefore the output value and all following processes can be affected by any proceeding processes and also the position in the chain.

---

## Main

### Log to feed

**Description:** This processor logs the current input value to a feed as a new datapoint at each occurrence of the set interval. 

**Argument:** The name of the feed you wish to write to, a feed will be created if it doesn't already exist, The required feed engine type and the sampling interval.

---

## Calibration

### x (scale)

**Description:** Multiply current process value by value given. This can be useful for both calibrating an input (eg x 0.95 to reduce by 5%), scaling (eg Watts to Kilowatts) and for retriving floats from integers received in place of floats (eg Vac 24000 x 0.01 = 240v). Can also be used to invert signed values (eg 3kW x -1 = -3kW).

**Argument:** The value to multiply (scale) by (can be negative).

### + (offset)

**Description:** Add the value given to the current process value. This can again be useful for calibration. Use a negative value to subtract.

**Argument:** The value to add (offset) or subtract (by using a negative value).

---

## Power

### Power to kWh

**Description:** Convert a current value given as power in Watts to a cumulative and ever rising plotted value in kWh. 

**Argument:** The feed name that you wish to write to. a feed can be created if it doesn't already exist, The required feed engine type and the sampling interval.

### Power to kWh/d

**Description:** This is a particularly useful input processor for energy monitoring, it converts a power value in Watts to a feed that contains an entry for the total energy used each day (kWh/d).

**Argument:** The feed name that you wish to write to. a feed will be created if it doesn't already exist.

### kWh to kWh/d

**Description:** Some datalogging equipment may just provide total kWh used or generated data, this can be used to convert this ever accumulating kWh variable to a kWh per day feed. (Note: in a pinch you can use this after an accumulator process to log the daily increase in the accumulator to a feed).

**Argument:** The feed name that you wish to write to. a feed can be created if it doesn't already exist. 

### kWh to Power

**Description:** This process will calculate a power value by deducting the last recorded kWh value from the current kWh value and dividing by the time elapsed.

**Argument:** The feed name that you wish to write to. a feed can be created if it doesn't already exist, The required feed engine type and the sampling interval.

### Wh increments to kWh/d

**Description:** Converts Wh (not kWh!) increment since last post data into a kWh per day feed. 

**Argument:** The feed name that you wish to write to. a feed can be created if it doesn't already exist.

### Histogram

**Description:** This converts power to energy vs power and plots data to a feed.

**Argument:** The feed name that you wish to write to. a feed can be created if it doesn't already exist.

---

## Input

### x input

**Description:** This multiplies the current process value by the current value of the selected input. 

**Argument:** Name of input to multiply by.

### / input

**Description:** This divides the current process value by the current value of the selected input.

**Argument:** Name of input to divide by.

### + input

**Description:** This adds the current value of the selected input to the current process value. 

**Argument:** Name of input to add.

### - input

**Description:** This subtracts the current value of the selected input from the current process value.

**Argument:** Name of input to subtract.

### Input on-time

**Description:** This counts the amount of time that an input is high in each day and logs to a feed. Created for counting the number of hours a solar hot water pump was on each day.

**Argument:** The feed name that you wish to write to. a feed can be created if it doesn't already exist.

---

## Misc

### Accumulator

**Description:** This adds the input value to the current value of the accumulator and logs it to a feed. The new total value of the feed is returned for further processing if required.

**Argument:** The feed name that you wish to write to. a feed can be created if it doesn't already exist, The required feed engine type and the sampling interval.

### Rate of change

**Description:** Display the rate of change for the current and last entry.

**Argument:** The feed name that you wish to write to. a feed can be created if it doesn't already exist, The required feed engine type and the sampling interval.

### Signed to Unsigned

**Description:** This will convert a signed integer (-32768 to 32767) to an unsigned integer (0 to 65535).

**Argument:** The feed value to be converted.

### Max value

**Description:** This will monitor the current process value and record a maximum value for each 24he period to a feed.

**Argument:** The name of the feed you wish to write to, a feed will be created if it doesn't already exist.

### Min value

**Description:**  This will monitor the current process value and record a minimum value for each 24he period to a feed.

**Argument:**  The name of the feed you wish to write to, a feed will be created if it doesn't already exist.

### Reset to Zero

**Description:** Used to reset the process chain by setting the value to ZERO, no previous processing steps will effect any steps after this point in the chain. Used in conjunction with either a + input or + feed to start another chain of processes. This will not cause a loss of data it only resets the calculation not any logged values.

**Argument:** none.

---

## Pulse

### Total pulse count to pulse increment 

**Description:** To be used where the input is the total number of pulses since some epoch (such as your meter's switch-on date) and does not reset each time the input is submitted to emonCMS. This process uses a feed to track the last value of the input, and passes on the change in input value only, which can be used for further processing. In the case that the input value has fallen, it is assumed that the pulse counter has rolled over, and the full value of the input is returned.

**Argument:** The feed name that you wish to use to track the last input value. If the feed does not yet exist, this will create a feed of the given name. (Note that the pulse increment value is not logged to feed by this processor - that must be done in a subsequent step.).

---

## Limits

### Allow Positive

**Description:** This will allow only positive values to pass and prevent any negative values passing. For instance when used on an electricity meter tail where PV is installed, this will allow import to be recorded without the value reducing when PV is being exported to the grid. Without this process only the nett import/export value could be determined.

**Argument:** none.

### Allow Negative

**Description:** This will allow only negative values to pass and prevent any positive values passing. Exactly the same as allow positive in operation but for negative values, very useful to record export.

**Argument:** none.

---

## Feed

### + feed

**Description:** Add the current value of the feed selected to the current process value.

**Argument:** The feed to Add.

### - feed

**Description:** Subtract the current value of the feed selected from the current process value.

**Argument:** The feed to subtract.

### x feed

**Description:** Multiply the current process value by the value of the feed selected. 

**Argument:** The feed to multiply by.

### / feed

**Description:** Divide the current process value by the current value of the feed selected.

**Argument:** The feed to divide by.



---

---



### phaseshift

**Description:** not currently supported.

**Argument:**

### update feed @ time

**Description:** not currently supported.

**Argument:** 

### average

**Description:** not currently supported.

**Argument:**
