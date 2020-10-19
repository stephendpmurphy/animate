# animate
A component animation library implemented in plain C. Animate allows a developer to apply a "vector" based animation to coordinate based components. This could be UI elements like addressable LEDs or Displays. Using a display as an example, the developer typically creates UI elements with their favorite library (I will be using u8g2 in these examples) and places the elements at a fixed location on the screen. (x=15, y=15 in this example)
```c
void display_name(void) {
    u8g2_FirstPage(&u8g2);
    do
    {
        u8g2_SetFont(&u8g2, u8g2_font_7x13B_tf);
        u8g2_DrawStr(&u8g2, 15, 15, "stephendpmurphy");
    } while (u8g2_NextPage(&u8g2));
}

```

The animate library can be used to apply an offset to the X,Y coordinates that increments/decrements at a configurable velocity from point A to point B. As an example you can implement a transition where the text enters the screen from the left by giving the transition a start point the following parameters
- X_start = -128
- X_end = 0
- velocity = 20

Once the transition is started it will begin the X coordinate at the specified start position and increment by 20 until the specified end position is reached. It is ***vitally important*** that the transition is "ticked" or updated at a fixed rate which matches the update rate of your display. A commented example is seen below where the text is brought in from the left of the screen.
```c
#include <stdlib.h>
#include "u8g2.h"
#include "animation.h"

u8g2_t u8g2;
animation_inst_t myAnimation = {0};
    
int maint(void) {

    int16_t X_OFFS_myAnimation;
    int16_t X_OFFS_myOtherAnimation;

    // Set our config to have a negative X offset
    // Pushing the UI element off the screen to the left.
    // As it ticks to zero the offset will decrease which brings
    // the UI element onto the screen, entering from the left.
    myAnimation.config.start.x = -128;
    myAnimation.config.end.x = 0;
    myAnimation.config.velocity.x = 20;

    // Begin the animation
    animation_start(&animation);

    while(1) {

        // Update myAnimation. This must "tick"
        // at a fixed update rate which matches
        // the display update rate
        animation_update(&myAnimation);

        // Get the current value of the animation X coordinate
        // and store it as an offset to be applied to our 
        // UI element
        X_OFFS_myAnimation = animation_getCurrentX(&myAnimation);

        // Update the display
        u8g2_FirstPage(&u8g2);
        do
        {
            u8g2_SetFont(&u8g2, u8g2_font_7x13B_tf);
            // Draw our UI element, and apply our X_OFFS from the 
            // animation
            u8g2_DrawStr(&u8g2, X_OFFS_myAnimation + 15, 15, "stephendpmurphy");
        } while (u8g2_NextPage(&u8g2));

        // Delay for 30ms giving us roughly a 33hz display update rate
        delay_ms(30);
    }
}
```

These animations can be chained via a "callback" that is executed when an animation reaches it's specified X,Y end coordinates. The callback takes no parameters and returns void. An example of beginning an animation at the completion of another is seen below. It is mostly the same as the example above, however, once the first UI element finishes and reaches the end X,Y coordinates the callback will fire which begins the next animation.
```c
#include <stdlib.h>
#include "u8g2.h"
#include "animation.h"

u8g2_t u8g2;
animation_inst_t myAnimation = {0};
animation_inst_t myOtherAnimation = {0};

void CB_animationComplete(void) {
    // Begin the animation
    animation_start(&myOtherAnimation);
}

int maint(void) {

    int16_t X_OFFS_myAnimation;
    int16_t X_OFFS_myOtherAnimation;

    // Set our config to have a negative X offset
    // Pushing the UI element off the screen to the left.
    // As it ticks to zero the offset will decrease which brings
    // the UI element onto the screen, entering from the left.
    myAnimation.config.start.x = -128;
    myAnimation.config.end.x = 0;
    myAnimation.config.velocity.x = 20;

    // Configure the other animation the same as above
    myOtherAnimation.config.start.x = -128;
    myOtherAnimation.config.end.x = 0;
    myOtherAnimation.config.velocity.x = 20;

    // Add our callback which will fire when the animation completes
    // thus starting the next animation
    myAnimation.cb = CB_animationComplete;

    // Begin the animation
    animation_start(&myAnimation);

    while(1) {
        // Update myAnimation. This must "tick"
        // at a fixed update rate which matches
        // the display update rate
        animation_update(&myAnimation);

        // Update myOtherAnimation. This does nothing until
        // the animation_start API is called. Until the animation
        // is started, the returned X,Y values will be the configured
        // start X,Y coordinates
        animation_update(&X_OFFS_myOtherAnimation);

        // Get the current value of the animation X coordinate
        // and store it as an offset to be applied to our
        // UI element
        X_OFFS_myAnimation = animation_getCurrentX(&myAnimation);
        // This will return the configured start X value until the animation
        // is started
        X_OFFS_myOtherAnimation = animation_getCurrentX(&X_OFFS_myOtherAnimation);

        // Update the display
        u8g2_FirstPage(&u8g2);
        do
        {
            u8g2_SetFont(&u8g2, u8g2_font_7x13B_tf);
            // Draw our UI element, and apply our X_OFFS_myAnimation from the
            // animation
            u8g2_DrawStr(&u8g2, X_OFFS_myAnimation + 15, 15, "stephendpmurphy");
            // Draw our second UI element, and apply our X_OFFS_myOtherAnimation from the
            // animation
            u8g2_DrawStr(&u8g2, X_OFFS_myAnimation + 15, 29, "animations!");
        } while (u8g2_NextPage(&u8g2));

        // Delay for 30ms giving us roughly a 33hz display update rate
        delay_ms(30);
    }
}
```
