using System.Runtime.InteropServices;

namespace ColorTriggerBot
{
    public partial class Form1 : Form
    {
        [DllImport("user32.dll")]
        static extern short GetAsyncKeyState(Keys vKey);

        [DllImport("user32.dll")]
        static extern void mouse_event(uint dwFlags, int x, int y, uint dwData, IntPtr dwExtraInfo);

        const uint LEFTDOWN = 0x02;
        const uint LEFTUP = 0x04;

        Keys hotkey = Keys.RShiftKey;
        bool hasClicked = false; // Flag to track if a click was performed

        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            label1.Text = "Hotkey is ->" + hotkey.ToString() + " <-";
            Thread th = new Thread(BackgroundLogic) { IsBackground = true };
            th.Start();
        }

        void BackgroundLogic()
        {
            while (true)
            {
                // If the hotkey (Shift) is being pressed
                if (KeyPressed(hotkey))
                {
                    // Check if red is detected
                    if (ColorSearch(ConvertIntToColor(0x890a0a), 904, 534, 981, 607))
                    {
                        // If red is detected and hasn't clicked yet
                        if (!hasClicked)
                        {
                            PerformClick(0, 0); // Perform the click
                            hasClicked = true; // Set the flag to indicate a click has been performed
                            Thread.Sleep(10); // Delay for 10 milliseconds before resetting the cycle
                        }
                    }
                    else
                    {
                        // Reset the click state if the color is no longer detected
                        hasClicked = false; // Allow another click when the color reappears
                    }
                }
                Thread.Sleep(30); // Short sleep to avoid CPU overload
            }
        }

        Color ConvertIntToColor(int intColor)
        {
            int red = (intColor >> 16) & 0xFF;
            int green = (intColor >> 8) & 0xFF;
            int blue = intColor & 0xFF;
            return Color.FromArgb(red, green, blue);
        }

        bool ColorSearch(Color wantedColor, int x1, int y1, int x2, int y2)
        {
            using (Bitmap screenshot = new Bitmap(x2 - x1, y2 - y1))
            {
                using (Graphics g = Graphics.FromImage(screenshot))
                {
                    g.CopyFromScreen(new Point(x1, y1), Point.Empty, screenshot.Size);

                    for (int x = 0; x < screenshot.Width; x++)
                    {
                        for (int y = 0; y < screenshot.Height; y++)
                        {
                            Color pixelColor = screenshot.GetPixel(x, y);

                            if (AreColorsClose(wantedColor, pixelColor, 0))
                            {
                                return true;
                            }
                        }
                    }
                    return false;
                }
            }
        }

        bool AreColorsClose(Color color1, Color color2, int maxColorDifference)
        {
            int redDiff = Math.Abs(color1.R - color2.R);
            int greenDiff = Math.Abs(color1.G - color2.G);
            int blueDiff = Math.Abs(color1.B - color2.B);
            return redDiff <= maxColorDifference && greenDiff <= maxColorDifference && blueDiff <= maxColorDifference;
        }

        void PerformClick(int x, int y)
        {
            mouse_event(LEFTDOWN, x, y, 0, IntPtr.Zero);
            Thread.Sleep(30); // Time for the mouse click
            mouse_event(LEFTUP, x, y, 0, IntPtr.Zero);
        }

        bool KeyPressed(Keys vKey)
        {
            return GetAsyncKeyState(vKey) < 0;
        }
    }
}

