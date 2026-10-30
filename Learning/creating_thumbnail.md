Blogs use images in 16:9 aspect ratio. use the tool below to resize the image:
https://imagy.app/change-image-to-16x9-aspect-ratio/

in excalidraw, when you select the components to make an image out of, the app leaves almost no padding. So the components span across the entire viewport. to avoid that draw a box around the components and then select that box as well. like this. the box ends up touching the edges of viewport giving the appearance of internal padding.

project logo: 
use figma for tilted stack look:

That reference image is helpful! It shows a **contained 16:9 "Hero" shot** with a nice background and a single 3D-tilted element.

Since you are in Figma, here is the exact workflow to create that "Stacked & Tilted" composite specifically for a **16:9 card**.

### Step 1: Create the Main Container (The 16:9 Frame)
1.  Press `F` (Frame tool).
2.  Click on the canvas.
3.  In the right sidebar (Design panel), set the dimensions to **1600 width** and **900 height**.
    *   *Note: This creates a high-res 16:9 aspect ratio. You can scale it down later, but design in high-res.*
4.  **Set the Background:**
    *   Since your code/terminal screenshots are **Dark Mode**, pick a background color for the frame that contrasts well.
    *   *Option A (Clean):* A soft off-white or light gray (like your reference) will make the dark cards pop.
    *   *Option B (Moody):* A deep, dark navy gradient (matching the loss graph) for a sleek "Cyberpunk/AI" look.

### Step 2: Prepare the 3 "Cards"
*Do this outside the main frame first.*

**Step 1: Create your standard shape**
1. Press **`R`** on your keyboard to select the Rectangle tool.
2. Click and drag on your canvas to draw a rectangle. 
3. In the **right sidebar**, set the exact dimensions you want. A good standard size for portfolio cards is **W: 800** and **H: 530**.
4. *(Optional but recommended)* Give it rounded corners! In the right sidebar, look for the little arc icon (Corner Radius) and set it to **16** or **24**.

**Step 2: Duplicate it**
1. Select your new rectangle.
2. Press **`Ctrl + D`** (or `Cmd + D` on Mac) twice so you have **3 identical rectangles** next to each other.

**Step 3: Put your images inside the rectangles**
1. Select the **first rectangle**.
2. Look at the right sidebar under the **"Fill"** section. You will see a colored square (usually gray). Click that square.
3. At the top of the pop-up menu, change **"Solid"** to **"Image"**.
4. Click the checkerboard area that says **"Choose image"** and upload your first screenshot (e.g., the Code).
5. Repeat this for the other two rectangles with your Graph and Illustration images.

**Step 4: Adjust the Crop (Crucial!)**
Because your code screenshot is tall and your illustration is wide, Figma will automatically try to center them.
1. If an image is cutting off an important part (like your PyTorch code), click the image fill box again in the right sidebar.
2. Change the dropdown menu from **"Fill"** to **"Crop"**. 
3. Now you can click and drag the image around *inside* the rectangle to frame the exact part of the code or graph you want to show! Press `Enter` when it looks good.

1.  **Create 3 Rectangles:** Make them **600 width x 400 height**. (A 3:2 aspect ratio works best for this stack).
2.  **Round the Corners:** Set "Corner Radius" to **20**.
3.  **Fill Images:** Use the "Rectangle Method" I shared earlier to put your Code, Graph, and Illustration into these 3 rectangles.
4.  **Add your Overlays (Do this BEFORE tilting):**
    *   **Card 1 (Code):** Paste a transparent PNG of the **PyTorch logo** into this frame. Align it to the corner.
    *   **Card 2 (Training):** Paste the **Node Graph icon** or overlay the terminal text here.
    *   **Card 3 (Illustration):** Draw a simple rounded rectangle on top, make it white, and type "Input: Once upon a time..." to simulate the chat interface. Group it with the illustration image.

### Step 3: The "Tilted Stack" Effect (Using Plugins)
Figma doesn't have a native "3D Rotate" tool in the sidebar yet, so you need a free plugin.

1.  **Select all 3 cards.**
2.  Go to **Resources** (top bar) > **Plugins** > Search **"SkewDat"**.
3.  **Run SkewDat.**
4.  Adjust the **"Vertical Skew"** to roughly **-15°** (or whatever angle looks like it's leaning back slightly).
5.  *Optional:* You can also add a slight rotation (like -5°) in the standard Figma sidebar to give it a dynamic angle.

### Step 4: Arrange and Depth
Now move these skewed cards into your main **1600x900 Frame**.

1.  **Positioning:**
    *   **Back Layer:** Place the **Code Card** on the left.
    *   **Middle Layer:** Place the **Training Card** in the center, overlapping the code card by about 30%. Move it slightly *down* so it looks closer.
    *   **Front Layer:** Place the **Illustration Card** on the right, overlapping the middle card. Move it further down.
2.  **Drop Shadows (Crucial for 3D):**
    *   Select all 3 cards.
    *   Go to **Effects** > **Drop Shadow**.
    *   **Blur:** 50
    *   **Y:** 30
    *   **Opacity:** 40%
    *   *Pro Tip:* Make the shadow on the "Front" card slightly larger/darker than the back one to emphasize depth.

### Step 5: Final Composition
1.  Select the whole group of cards.
2.  Center them visually in your 1600x900 frame.
3.  Ensure there is "breathing room" (padding) around the edges so the cards don't touch the borders of the 16:9 frame.

### Step 6: Export
1.  Click on the **Main 1600x900 Frame** name.
2.  Scroll to the bottom of the right sidebar to **Export**.
3.  Choose **PNG**.
4.  Click **Export [Frame Name]**.

You now have a perfectly sized 16:9 image that fits your website's project card slot!


To match that exact "Warm Sunset" aesthetic from your neighbor card, you need a subtle vertical linear gradient that goes from a **Soft Cream** to a **Warm Peach**.

Here are the hex codes and the Figma recipe to recreate it perfectly:

### The "Warm Portfolio" Gradient Recipe

*   **Type:** Linear Gradient
*   **Direction:** Vertical (Top ↓ Bottom)
*   **Top Color (0%):** `#FFF2DB` (Very pale cream/orange)
*   **Bottom Color (100%):** `#FFD8AB` (Warm apricot/peach)

### How to Apply this in Figma

1.  Select your main **1600x900 Frame** (the background container).
2.  Go to the **Fill** section in the right sidebar.
3.  Click the colored square to open the color picker.
4.  At the top left of the popup, change **"Solid"** to **"Linear"**.
5.  **Set the Top Stop:**
    *   Click the left square on the gradient bar.
    *   Paste this Hex: **`#FFF2DB`**
    *   Ensure Opacity is **100%**.
6.  **Set the Bottom Stop:**
    *   Click the right square on the gradient bar.
    *   Paste this Hex: **`#FFD8AB`**
    *   Ensure Opacity is **100%**.

### Why this works
This gradient replicates the lighting in your reference image. The lighter top simulates an overhead light source, and the warmer bottom grounds the image. Since your illustration of the mother and children has warm, cozy lighting (lamp light), this background will blend seamlessly with it while contrasting nicely against the dark code terminal.