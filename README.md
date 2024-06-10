using System;
using System.Threading.Tasks;

namespace SimpleWebCrawler
{
    class Program
    {
        static async Task Main(string[] args)
        {
            string startUrl = "https://google.com"; // Change this to the website you want to crawl
            string outputDirectory = "C:\\Users\\User\\Desktop"; // Change this to the output directory

            var webCrawler = new WebCrawler(startUrl, outputDirectory);
            await webCrawler.Crawl();

            Console.WriteLine("Crawling completed. Press any key to exit.");
            Console.ReadKey();
        }
    }
}









using HtmlAgilityPack;
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using System.Threading.Tasks;

namespace SimpleWebCrawler
{
    public class WebCrawler
    {
        private readonly string _startUrl;
        private readonly string _outputDirectory;
        private readonly int _maxDepth;
        private HashSet<string> _visitedUrls;

        public WebCrawler(string startUrl, string outputDirectory, int maxDepth = 3)
        {
            _startUrl = startUrl;
            _outputDirectory = outputDirectory;
            _maxDepth = maxDepth;
            _visitedUrls = new HashSet<string>();
        }

        public async Task Crawl()
        {
            // Create output directory if it doesn't exist
            Directory.CreateDirectory(_outputDirectory);

            await CrawlPage(_startUrl, depth: 0);
        }

        private async Task CrawlPage(string url, int depth)
        {
            try
            {
                if (depth > _maxDepth || _visitedUrls.Contains(url))
                {
                    return;
                }

                var httpClient = new HttpClient();
                var html = await httpClient.GetStringAsync(url);

                var htmlDocument = new HtmlDocument();
                htmlDocument.LoadHtml(html);

                // Save the page as a static file
                string fileName = Path.Combine(_outputDirectory, GetSafeFilename(url) + ".html");
                File.WriteAllText(fileName, html);

                Console.WriteLine($"Page saved: {url}");
                _visitedUrls.Add(url);

                // Find and crawl links on the page
                var links = htmlDocument.DocumentNode.SelectNodes("//a[@href]");
                if (links != null)
                {
                    foreach (var link in links)
                    {
                        string href = link.GetAttributeValue("href", "");
                        if (!string.IsNullOrWhiteSpace(href))
                        {
                            // Normalize the URL if needed and crawl recursively
                            string absoluteUri = new Uri(new Uri(url), href).AbsoluteUri;
                            await CrawlPage(absoluteUri, depth + 1);
                        }
                    }
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error crawling {url}: {ex.Message}");
            }
        }

        private string GetSafeFilename(string filename)
        {
            return string.Join("_", filename.Split(Path.GetInvalidFileNameChars()));
        }
    }
}
