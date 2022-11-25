---
layout: post
title: Efficient way to remove ALL whitespace from String?
description: Fastest and most efficient way to remove ALL whitespace from String?
categories: [programming, c#, dotnet, optimization]
---

I have had a [StackOverflow answer](https://stackoverflow.com/questions/6219454/efficient-way-to-remove-all-whitespace-from-string/37347881#37347881) to the question "Efficient way to remove ALL whitespace from String?" for a while now. After [.NET 7 was released](https://github.com/dotnet/core/blob/main/release-notes/7.0/7.0.0/7.0.0.md?WT.mc_id=dotnet-35129-website) with many performance optimizations I figured I wanted to go over this answer once again testing all of the "remove whitespace" methods up against multiple frameworks. To make the test a little more robust I used [BenchmarkDotnet](https://benchmarkdotnet.org/) library. 

The runtimes I added to this test was 
- .NETcore 3.1
- .NET 6
- .NET 7

# Benchmark code
```C#
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;

using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Jobs;

namespace RemoveWhitespaceBenchmark
{
    [SimpleJob(RuntimeMoniker.NetCoreApp31)]
    [SimpleJob(RuntimeMoniker.Net60)]
    [SimpleJob(RuntimeMoniker.Net70)]
    public class RemoveWhitespace
    {
        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveStringReader(string text, string name)
        {
            StringBuilder s = new StringBuilder(text.Length);
            using (StringReader reader = new StringReader(text))
            {
                int i = 0;
                char c;
                for (; i < text.Length; i++)
                {
                    c = (char)reader.Read();
                    if (!char.IsWhiteSpace(c))
                    {
                        s.Append(c);
                    }
                }
            }

            return s.ToString();
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveLinqNativeCharIsWhitespace(string text, string name)
        {
            return new string(text.ToCharArray()
                .Where(c => !char.IsWhiteSpace(c))
                .ToArray());
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveLinq(string text, string name)
        {
            return new string(text.ToCharArray()
                .Where(c => !Char.IsWhiteSpace(c))
                .ToArray());
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveRegex(string text, string name)
        {
            return Regex.Replace(text, @"\s+", "");
        }

        private static readonly Regex compiled = new Regex(@"\s+", RegexOptions.Compiled);

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveRegexCompiled(string text, string name)
        {
            return compiled.Replace(text, "");
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveForLoop(string text, string name)
        {
            string input = text;
            for (int i = input.Length - 1; i >= 0; i--)
            {
                if (char.IsWhiteSpace(input[i]))
                {
                    input = input.Remove(i, 1);
                }
            }
            return input;
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string StringSplitThenJoin(string text, string name)
        {
            return string.Join("", text.Split(default(string[]), StringSplitOptions.RemoveEmptyEntries));
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveInPlaceCharArray(string text, string name)
        {
            int len = text.Length;
            char[] src = text.ToCharArray();
            int dstIdx = 0;
            for (int i = 0; i < len; i++)
            {
                char ch = src[i];
                switch (ch)
                {
                    case '\u0020':
                    case '\u00A0':
                    case '\u1680':
                    case '\u2000':
                    case '\u2001':
                    case '\u2002':
                    case '\u2003':
                    case '\u2004':
                    case '\u2005':
                    case '\u2006':
                    case '\u2007':
                    case '\u2008':
                    case '\u2009':
                    case '\u200A':
                    case '\u202F':
                    case '\u205F':
                    case '\u3000':
                    case '\u2028':
                    case '\u2029':
                    case '\u0009':
                    case '\u000A':
                    case '\u000B':
                    case '\u000C':
                    case '\u000D':
                    case '\u0085':
                        continue;
                    default:
                        src[dstIdx++] = ch;
                        break;
                }
            }
            return new string(src, 0, dstIdx);
        }

        // Short input
        private const string SHORT_TEXT = "123 123 \t 1adc \n 222";

        private const string EXPECTED_SHORT_TEXT = "1231231adc222";

        // Long input
        private const string LONG_TEXT = "123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222";

        private const string EXPECTED_LONG_TEXT = "1231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc222";

        public IEnumerable<object[]> Data()
        {
            yield return new object[] { SHORT_TEXT, "SHORT" };
            yield return new object[] { LONG_TEXT, "LONG" };
        }
    }
}
```

*Important:* The `.csproj` file has to look something like this: 
```XML
<PropertyGroup>
  <TargetFrameworks>netcoreapp3.1;net6.0;net7.0</TargetFrameworks>
  <ImplicitUsings>disable</ImplicitUsings>
  <GenerateProgramFile>false</GenerateProgramFile>
  [...]
</PropertyGroup>
```
Since it is older versions of C#, implicit usings is not a feature.

# Running the benchmark test
To run the benchmark you only have to add this to your `Program.cs` file
```C#
public static void Main(string[] args)
{
    BenchmarkRunner.Run<RemoveWhitespace>();
}
```

# Results
## Short Text
| Method                           | Job           | Runtime       | name  | Mean     | Error  | StdDev | Median   |
| -------------------------------- | ------------- | ------------- | ----- | -------- | ------ | ------ | -------- |
| RemoveForLoop                    | .NET Core 3.1 | .NET Core 3.1 | SHORT | 246.68   | 4.885  | 5.999  | 244.6    |
| RemoveForLoop                    | .NET 7.0      | .NET 7.0      | SHORT | 314.28   | 6.207  | 8.902  | 313.59   |
| RemoveForLoop                    | .NET 6.0      | .NET 6.0      | SHORT | 505.46   | 9.457  | 8.846  | 501.08   |
| RemoveInPlaceCharArray           | .NET 6.0      | .NET 6.0      | SHORT | 56.17    | 1.601  | 4.516  | 55.45    |
| RemoveInPlaceCharArray           | .NET 7.0      | .NET 7.0      | SHORT | 59.29    | 1.221  | 1.751  | 58.68    |
| RemoveInPlaceCharArray           | .NET 6.0      | .NET 6.0      | SHORT | 122.86   | 2.435  | 2.158  | 122.7    |
| RemoveLinq                       | .NET 6.0      | .NET 6.0      | SHORT | 195.59   | 3.962  | 6.05   | 195.68   |
| RemoveLinq                       | .NET 7.0      | .NET 7.0      | SHORT | 205.01   | 4.172  | 9.502  | 203.98   |
| RemoveLinq                       | .NET 6.0      | .NET 6.0      | SHORT | 208.75   | 6.48   | 17.74  | 204.15   |
| RemoveLinqNativeCharIsWhitespace | .NET 7.0      | .NET 7.0      | SHORT | 158.74   | 5.834  | 17.11  | 152.9    |
| RemoveLinqNativeCharIsWhitespace | .NET 7.0      | .NET 7.0      | SHORT | 202.84   | 4.001  | 3.742  | 201.46   |
| RemoveLinqNativeCharIsWhitespace | .NET Core 3.1 | .NET Core 3.1 | SHORT | 238.61   | 3.902  | 3.258  | 237.57   |
| RemoveRegex                      | .NET 7.0      | .NET 7.0      | SHORT | 96.39    | 2.538  | 7.282  | 95.44    |
| RemoveRegex                      | .NET 6.0      | .NET 6.0      | SHORT | 145      | 6.99   | 19.829 | 138.66   |
| RemoveRegex                      | .NET Core 3.1 | .NET Core 3.1 | SHORT | 1,007.07 | 19.556 | 25.429 | 1,004.26 |
| RemoveRegexCompiled              | .NET Core 3.1 | .NET Core 3.1 | SHORT | 155.61   | 3.025  | 3.483  | 154.43   |
| RemoveRegexCompiled              | .NET Core 3.1 | .NET Core 3.1 | SHORT | 156.72   | 3.024  | 3.361  | 156.18   |
| RemoveRegexCompiled              | .NET Core 3.1 | .NET Core 3.1 | SHORT | 1,199.94 | 22.786 | 22.379 | 1,189.29 |
| RemoveStringReader               | .NET Core 3.1 | .NET Core 3.1 | SHORT | 53.05    | 1.105  | 1.085  | 52.72    |
| RemoveStringReader               | .NET 6.0      | .NET 6.0      | SHORT | 275.37   | 4.94   | 5.49   | 274.29   |
| RemoveStringReader               | .NET 7.0      | .NET 7.0      | SHORT | 550.95   | 9.963  | 8.832  | 548.1    |
| StringSplitThenJoin              | .NET 6.0      | .NET 6.0      | SHORT | 196.58   | 3.864  | 4.134  | 195.19   |
| StringSplitThenJoin              | .NET 7.0      | .NET 7.0      | SHORT | 214.77   | 5.211  | 15.035 | 209.56   |
| StringSplitThenJoin              | .NET Core 3.1 | .NET Core 3.1 | SHORT | 246.52   | 2.866  | 2.541  | 247.07   |

## Long Text
|----------------------------------+---------------+---------------+------+-----------+---------+---------+-----------|
| Method                           | Job           | Runtime       | name | Mean      | Error   | StdDev  | Median    |
|----------------------------------|---------------|---------------|------|-----------|---------|---------|-----------|
| RemoveForLoop                    | .NET 7.0      | .NET 7.0      | LONG | 9,534.81  | 187.211 | 229.912 | 9,450.11  |
| RemoveForLoop                    | .NET 6.0      | .NET 6.0      | LONG | 9,574.58  | 170.786 | 250.336 | 9,560.20  |
| RemoveForLoop                    | .NET Core 3.1 | .NET Core 3.1 | LONG | 10,212.44 | 200.69  | 187.725 | 10,218.33 |
| RemoveInPlaceCharArray           | .NET 6.0      | .NET 6.0      | LONG | 795.89    | 15.824  | 26      | 789.73    |
| RemoveInPlaceCharArray           | .NET 7.0      | .NET 7.0      | LONG | 860.86    | 17.298  | 26.416  | 851.84    |
| RemoveInPlaceCharArray           | .NET Core 3.1 | .NET Core 3.1 | LONG | 865.46    | 29.865  | 85.206  | 842.39    |
| RemoveLinq                       | .NET 7.0      | .NET 7.0      | LONG | 2,127.38  | 38.694  | 48.936  | 2,110.64  |
| RemoveLinq                       | .NET 6.0      | .NET 6.0      | LONG | 2,131.09  | 38.736  | 54.302  | 2,113.29  |
| RemoveLinq                       | .NET Core 3.1 | .NET Core 3.1 | LONG | 2,467.73  | 31.776  | 24.809  | 2,475.30  |
| RemoveLinqNativeCharIsWhitespace | .NET 6.0      | .NET 6.0      | LONG | 2,090.81  | 19.523  | 15.242  | 2,085.87  |
| RemoveLinqNativeCharIsWhitespace | .NET 7.0      | .NET 7.0      | LONG | 2,162.54  | 40.383  | 49.594  | 2,144.46  |
| RemoveLinqNativeCharIsWhitespace | .NET Core 3.1 | .NET Core 3.1 | LONG | 2,251.41  | 43.37   | 46.405  | 2,234.34  |
| RemoveRegex                      | .NET 6.0      | .NET 6.0      | LONG | 8,029.84  | 148.297 | 123.835 | 8,014.60  |
| RemoveRegex                      | .NET 7.0      | .NET 7.0      | LONG | 8,064.66  | 153.11  | 150.375 | 8,044.91  |
| RemoveRegex                      | .NET Core 3.1 | .NET Core 3.1 | LONG | 23,199.52 | 297.498 | 232.267 | 23,290.44 |
| RemoveRegexCompiled              | .NET 6.0      | .NET 6.0      | LONG | 3,673.86  | 58.893  | 49.179  | 3,665.01  |
| RemoveRegexCompiled              | .NET 7.0      | .NET 7.0      | LONG | 3,858.84  | 74.988  | 80.236  | 3,834.91  |
| RemoveRegexCompiled              | .NET Core 3.1 | .NET Core 3.1 | LONG | 20,208.25 | 302.477 | 268.138 | 20,191.36 |
| RemoveStringReader               | .NET 7.0      | .NET 7.0      | LONG | 1,239.48  | 22.25   | 23.807  | 1,234.84  |
| RemoveStringReader               | .NET 6.0      | .NET 6.0      | LONG | 2,207.21  | 42.806  | 57.144  | 2,186.48  |
| RemoveStringReader               | .NET Core 3.1 | .NET Core 3.1 | LONG | 2,925.74  | 54.859  | 106.999 | 2,893.67  |
| StringSplitThenJoin              | .NET 7.0      | .NET 7.0      | LONG | 2,860.38  | 48.188  | 42.717  | 2,849.34  |
| StringSplitThenJoin              | .NET 6.0      | .NET 6.0      | LONG | 2,872.59  | 56.242  | 75.082  | 2,846.50  |
| StringSplitThenJoin              | .NET Core 3.1 | .NET Core 3.1 | LONG | 3,833.31  | 59.144  | 63.283  | 3,844.23  |
|----------------------------------+---------------+---------------+------+-----------+---------+---------+-----------|

|test|table|
|---|---|
|content|2|
