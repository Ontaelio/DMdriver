# DMdriver
An Arduino library for DM63x LED driver chips
(c) 2016 Dmitry Reznkov

Full instructions can be found at Instructables.com:
http://www.instructables.com/id/Hundreds-of-LEDs-on-Arduino-a-New-Way-From-the-Pas

Excerpt:

The library is compatible with any chip in the DM63x line. In fact, you can use different ones in the same project, provided they are connected independently (not chained, different latch pins). It does all the basic stuff you’ll expect from such a library, leaving you to worry about what you want to achieve, not how to achieve that.

First, declare an object of the DMdriver type called, for example, TestObject:

DMdriver TestObject (DM634, 3, 9);

On declaration, you have to provide three values. First, the type of the driver you have – it can be DM631, DM632, DM633 or DM634. Secondly, the number of LED drivers chained in this particular object. Lastly, the pin connected to the latch line of this object.
You can declare multiple objects connected to different latch pins. For example, the contraption in the video and an illustration above has 4 such objects running on a mix of DM633 and DM634 chips. You can also declare the same physical object connected to one latch pin multiple times as different virtual objects if needed, just note that it will allocate more video memory (however, the actual memory allocation won’t happen at this step).

To use the DMdriver object you’ll have to initialize it once in the setup() section:

TestObject.init();

This will set up the SPI interface and allocate the needed memory to your object. The memory will be allocated as a dynamic array, this is the weird way of C++. If you initialize all your objects during setup() and never try to re-allocate their memory later that won’t be a problem.

Now, if you do the init() with no parameters, the LEDs connected will be addressed by the output pins on LED drivers, meaning 0 for the first one, 1 for the second, …, 16 for the first pin on the second chained chip and so on. Sometimes when you are designing your own board, especially if you are making it yourself, you won’t be able to install your LEDs in this particular order. This is especially cumbersome with RGB LEDs: your first RGB LED may, in fact, be connected to outputs 0, 4 and 6, the second to 1, 3 and 12 and so on. The init() function can take an optional parameter pointing to an array of values representing the physical LED connections, a look-up table.

The look-up table is declared as an array of bytes representing the string of physical LED driver pins according to the order you want them to be in. In the aforementioned case, this array should begin like this:

byte orderLED[] = {0, 4, 6, 1, 3, 12, …… }

and contain the number of bytes according to the number of all the outputs in the object, in this example 48. It should be declared before the setup() section, obviously, and now you’ll initialize the TestObject like this:

TestObject.init(orderLED);

There are also two destructors that will free the memory used by both arrays, but you should never use them.

Now that we have our object declared and initialized, we can start using it. As explained before, we will be building a needed picture in the ‘video memory’ and then sending it in full for the LED driver to display. First,

TestObject.clearAll();

fills the video memory with zeros. 

TestObject.setPoint(num, value);

sets the individual LED at address num to value. If the look-up table was provided during initialization, the library will get the actual pin number from it. For now, the change happens in memory only. 

value = TestObject.getPoint(num);

Returns the current value of the LED #num. Note that this value is also from memory, so it won’t necessarily correspond to the actually visible color.

TestObject.setRGBpoint(num, red, green, blue);

sets the RGB LED num to the color corresponding to red, green and blue values. The num is the position of the RGB LED in your chain of RGB LEDs (starting with 0). This will set red value to the output calculated as num*3, green to num*3+1 and blue to num*3+2; it is assumed that you have either connected your RGB LEDs in the right order or used a look-up table as described before.

There are two other ways of setting up an RGB LED. Firstly,

TestObject.setRGBled(num, red, green, blue);

sets up the RGB LED that has it’s red cathode connected to output num, assuming that green is num+1 and blue is num+2. This is a tiny bit faster. Secondly,

TestObject.setRGBmax(num, red, green, blue <, max>);

works the same as the setRGBpoint() function, but has an optional max parameter. This parameter limits the overall brightness of the RGB LED; if the sum of red, green and blue exceeds max, the values will be reduced, keeping their proportions intact. This function is useful if you need to keep the power consumption in check; the idea is described in more detail in this instructable. It can also be used for brightness control and fade-out effects; just keep in mind that it is more complex than previous functions, so it eats a bit more memory and is a bit slower.

Note that you should use only one of these three setRGB functions in your sketch to save program memory. Just select the one that suits your needs best and stick to it. That’s why max is optional in setRGBmax().

Now that we have the needed values stored in video memory, we can do

TestObject.sendAll();

to actually display the result.

With DM633 and DM634 chips you can also set the global brightness using

TestObject.setGlobalBrightness(val);

where val is a 7-bit number from 0 to 127. Note that 0 doesn’t mean ‘shut off’, it more like ‘very weak’. It is better to always set the GBC just after the init(), especially with DM634 chips, as they tend to change it randomly on startup. You can also 

TestObject.setGBCbyDriver(byteArray);

to set the global brightness individually for each connected DM633 or DM634 chip. This is useful if you have separate drivers controlling red, green and blue colors in all the RGB LEDs, so you can do some color calibration. The byteArray here is an array of brightness values for each of the drivers, starting with the first one in chain.

A bit of configuration options can be found in DMconfig.h.
