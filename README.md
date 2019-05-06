# FFMPEG Stream Input Image Set

This C# script is helpful when you use FFMPEG on Windows (cannot use glob) and need to generate a video
from a series of image files which are not named with sequential numbering component.

You can run this script using `dotnet script script.csx`:

```cs
using System.Threading;

var processStartInfo = new ProcessStartInfo();
processStartInfo.FileName = "ffmpeg.exe";
processStartInfo.Arguments = "-y -f image2pipe -i - -vf scale=320:-1 streamed-scaled.jpg";
// Note that this starts a separate process
processStartInfo.UseShellExecute = false;
processStartInfo.RedirectStandardInput = true;
var process = Process.Start(processStartInfo);
// Load the file for streaming into the process
var bytes = File.ReadAllBytes("img.jpg");
await process.StandardInput.BaseStream.WriteAsync(bytes, 0, bytes.Length);
```

To stream in multiple files and generate a video: (Includes extra settings for iPhone compatibility)

```cs
// https://www.nuget.org/packages/System.Drawing.Common
#r "nuget: System.Drawing.Common, 4.5.1"

using System;
using System.Drawing;
using System.Drawing.Drawing2D;
using System.Drawing.Imaging;
using System.IO;
using System.Threading;

var processStartInfo = new ProcessStartInfo();
processStartInfo.FileName = "ffmpeg.exe";
processStartInfo.Arguments = "-y -f image2pipe -i - -profile:v baseline -level 3.0 streamed-generated-iphone-small.mp4";
// Note that this starts a separate process
processStartInfo.UseShellExecute = false;
processStartInfo.RedirectStandardInput = true;
var process = Process.Start(processStartInfo);

using (var memoryStream = new MemoryStream())
{
  using (var bitmap = new Bitmap(1280, 720))
  {
    using (var fontFamily = new FontFamily("Arial"))
    {
      using (var font = new Font(fontFamily, 120))
      {
        using (var graphics = Graphics.FromImage(bitmap))
        {
          for (var index = 0; index < 500; index++)
          {
            graphics.Clear(Color.White);
            graphics.SmoothingMode = SmoothingMode.AntiAlias;
            graphics.DrawEllipse(Pens.Black, 10, 10, 500, 500);
            graphics.DrawString("TEST " + index.ToString(), font, Brushes.Black, 50, 50);
            graphics.Save();
            bitmap.Save(memoryStream, ImageFormat.Jpeg);
            var bytes = memoryStream.ToArray();
            memoryStream.Seek(0, SeekOrigin.Begin);
            await process.StandardInput.BaseStream.WriteAsync(bytes, 0, bytes.Length);
          }
        }
      }
    }
  }
}
```
