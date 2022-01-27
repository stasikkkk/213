using System;
using Newtonsoft.Json;
using System.Net;
using System.IO;
using System.Collections.Generic;
using Telegram.Bot;
using Telegram.Bot.Args;
using System.Xml.Serialization;
using System.Xml;
using Telegram.Bot.Types.ReplyMarkups;

namespace _213
{
    public class Program
    {
        static string key = "5073168261:AAF3VWS7d4t09SVDG3u7T9uGjlNNqNrt6L0";
        private static TelegramBotClient client;
        static float tempOfCity;
        static string nameOfCity;
        static string answerOnWether;//ответ у погоды
        static string text_mess_save1;
        static int text_mess_save2;//сохраненное сообщение
        static float feels_like_temp;//ощущаемая температура
        static float speed_wind;//скорость ветра
        static string wind_direction;//направление ветра
        static int humidity;//влажность
        static List<string> title = new List<string>();
        static List<string> ssilka = new List<string>();
        static List<string> photo = new List<string>();
        static int count = 0;
        static string fox_url;
        static string dog_url;
        static void Main(string[] args)
        {
            client = new TelegramBotClient(key);
            client.StartReceiving();
            client.OnMessage += OnMessageHeand;
            Console.ReadLine();
            client.StopReceiving();
        }

        private static async void OnMessageHeand(object sender, MessageEventArgs e)
        {
            var msg = e.Message;
            if (msg.Text != null)
            {
                Console.WriteLine($"Сообщение пользователя: {msg.Text}");
                switch (msg.Text)
                {
                    case "Кек":
                        var stic = await client.SendStickerAsync(
                            chatId: msg.Chat.Id,
                            sticker: "https://cdn.tlgrm.app/stickers/dc7/a36/dc7a3659-1457-4506-9294-0d28f529bb0a/192/2.webp",
                            replyMarkup: GetBut()
                            );
                        news();
                        break;

                    case "☁ Погода":
                        await client.SendTextMessageAsync(
                            msg.Chat.Id,
                            "Введите город: ",
                            replyMarkup: GetBut2());
                        text_mess_save1 = msg.Text;
                        break;

                    case "💱 Валюта":
                        await client.SendTextMessageAsync(
                            msg.Chat.Id,
                            "Введите 3 буквы валюты: ",
                            replyMarkup: GetBut3());
                        text_mess_save1 = msg.Text;
                        break;

                    case "📰 Новости":
                        news();
                        await client.SendTextMessageAsync(
                            msg.Chat.Id,
                            $"Сколько вам нужно новостей? (до {count}) ",
                            replyMarkup: GetBut2());
                        text_mess_save1 = msg.Text;

                        break;

                    case "🐾 Случайное фото животного":
                        await client.SendTextMessageAsync(
                            msg.Chat.Id,
                            $"Выберите животное",
                            replyMarkup: GetBut4());
                        text_mess_save1 = msg.Text;
                        break;

                    case "🚪 Выход":
                        await client.SendTextMessageAsync(
                            msg.Chat.Id,
                            "Жду вашей команды",
                            replyMarkup: GetBut());
                        text_mess_save1 = msg.Text;
                        break;
                    default:
                        Random rnd = new Random();
                        if (text_mess_save1 == "☁ Погода")
                        {
                            Weather(msg.Text);
                            if (nameOfCity != null)
                            {
                                await client.SendTextMessageAsync(
                                msg.Chat.Id,
                                $"Погода в {msg.Text} cегодня {Math.Round(tempOfCity, 2)}°, ощущается как {Math.Round(feels_like_temp, 2)}°\n" +
                                $"Ветер {wind_direction} скорость {speed_wind} м/с.\n" +
                                $"Влажность воздуха {humidity}%\n" +
                                $"{answerOnWether}");
                                nameOfCity = null;
                            }
                            else
                            {
                                await client.SendPhotoAsync(
                                        chatId: msg.Chat.Id,
                                        photo: "https://http.cat/400",
                                        caption: $"Такого города не существует",
                                        replyMarkup: GetBut2());
                            }
                        }
                        else if (text_mess_save1 == "🚪 Выход")
                        {
                            await client.SendTextMessageAsync(
                                msg.Chat.Id,
                                "Нельзя выйти из меню выбора команд",
                                replyMarkup: GetBut());
                        }
                        else if (text_mess_save1 == "💱 Валюта")
                        {
                            List<CurrencyRate> tmp = GetExchangeRates();
                            string price;
                            if (msg.Text.Length == 3)
                            {
                                string valuta = msg.Text.ToUpper();
                                if (valuta != "RUB")
                                {
                                    price = tmp.FindLast(s => s.CurrencyStringCode == valuta).ExchangeRate.ToString();
                                }
                                else
                                {
                                    price = "1";
                                }
                                await client.SendTextMessageAsync(
                                msg.Chat.Id,
                                $"Курс {valuta} cегодня {price} RUB.");
                            }
                            else
                            {
                                await client.SendTextMessageAsync(
                                msg.Chat.Id,
                                "Такого не существует",
                                replyMarkup: GetBut3());
                            }
                        }
                        else if (text_mess_save1 == "📰 Новости")
                        {
                            text_mess_save2 = Convert.ToInt32(msg.Text);
                            if (text_mess_save2 > count)
                            {
                                await client.SendTextMessageAsync(
                                msg.Chat.Id,
                                "Столько новостей нет :(",
                                replyMarkup: GetBut());
                            }
                            else
                            {
                                for (int i = 0; i < text_mess_save2; i++)
                                {
                                    await client.SendPhotoAsync(
                                        chatId: msg.Chat.Id,
                                        photo: photo[i],
                                        caption: $"{title[i]}\n" +
                                        $"Ссылка: {ssilka[i]}",
                                        replyMarkup: GetBut());
                                }
                                count = 0;
                                ssilka.Clear();
                                photo.Clear();
                                title.Clear();
                            }
                        }
                        else if (text_mess_save1 == "🐾 Случайное фото животного")
                        {
                            if (msg.Text == "🦊 Лисичка")
                            {
                                animals_fox();
                                await client.SendPhotoAsync(
                                        chatId: msg.Chat.Id,
                                        photo: fox_url,
                                        replyMarkup: GetBut4());
                            }
                            if (msg.Text == "🐶 Собачка")
                            {
                                animals_dog();
                                await client.SendPhotoAsync(
                                        chatId: msg.Chat.Id,
                                        photo: dog_url,
                                        replyMarkup: GetBut4());
                            }
                            if (msg.Text == "🐱 Котик")
                            {
                                int temp = rnd.Next(0, 1677);
                                await client.SendPhotoAsync(
                                        chatId: msg.Chat.Id,
                                        photo: $"https://aws.random.cat/view/{temp}",
                                        replyMarkup: GetBut4());
                            }
                        }
                        else
                        {
                            await client.SendTextMessageAsync(
                                msg.Chat.Id,
                                "Здравствуйте, это бот новостей и погоды! Выберете команду из списка ниже",
                                replyMarkup: GetBut());
                        }
                        break;
                }
            }
        }

        private static IReplyMarkup GetBut()
        {
            return new ReplyKeyboardMarkup
            {
                Keyboard = new List<List<KeyboardButton>>
                {
                    new List<KeyboardButton>{ new KeyboardButton { Text = "☁ Погода" }, new KeyboardButton { Text = "💱 Валюта" }, new KeyboardButton { Text = "📰 Новости" }
                    , new KeyboardButton { Text = "🐾 Случайное фото животного" }},
                }
            };
        }

        private static IReplyMarkup GetBut2()
        {
            return new ReplyKeyboardMarkup
            {
                Keyboard = new List<List<KeyboardButton>>
                {
                    new List<KeyboardButton>{ new KeyboardButton { Text = "🚪 Выход" } },
                }
            };
        }
        private static IReplyMarkup GetBut3()
        {
            return new ReplyKeyboardMarkup
            {
                Keyboard = new List<List<KeyboardButton>>
                {
                    new List<KeyboardButton>{ new KeyboardButton { Text = "USD" }, new KeyboardButton { Text = "EUR" }, new KeyboardButton { Text = "GBP" } },
                    new List<KeyboardButton>{ new KeyboardButton { Text = "🚪 Выход" } }
                }
            };
        }
        private static IReplyMarkup GetBut4()
        {
            return new ReplyKeyboardMarkup
            {
                Keyboard = new List<List<KeyboardButton>>
                {
                    new List<KeyboardButton>{ new KeyboardButton { Text = "🐱 Котик" }, new KeyboardButton { Text = "🐶 Собачка" }, new KeyboardButton { Text = "🦊 Лисичка" } },
                    new List<KeyboardButton>{ new KeyboardButton { Text = "🚪 Выход" } }
                }
            };
        }

        public static void Weather(string cityName)
        {
            try
            {
                string url = $"https://api.openweathermap.org/data/2.5/weather?q={cityName}&unit=metric&appid=7252e610985de722305daca578912507";
                HttpWebRequest httpWebRequest = (HttpWebRequest)WebRequest.Create(url);
                HttpWebResponse httpWebResponse = (HttpWebResponse)httpWebRequest.GetResponse();
                string response;

                using (StreamReader streamReader = new StreamReader(httpWebResponse.GetResponseStream()))
                {
                    response = streamReader.ReadToEnd();
                }
                WeatherResponse weatherResponse = JsonConvert.DeserializeObject<WeatherResponse>(response);

                nameOfCity = weatherResponse.Name;
                tempOfCity = weatherResponse.Main.Temp - 273;
                feels_like_temp = weatherResponse.Main.Feels_like - 273;
                speed_wind = weatherResponse.wind.speed;
                int deg = weatherResponse.wind.deg;
                double temp = Math.Round(feels_like_temp, 2);
                humidity = weatherResponse.Main.humidity;

                if (deg > 0 && deg < 30)
                    wind_direction = "северный";
                else if (deg > 31 && deg < 60)
                    wind_direction = "северо-восточный";
                else if (deg > 61 && deg < 120)
                    wind_direction = "восточный";
                else if (deg > 121 && deg < 150)
                    wind_direction = "юго-восточный";
                else if (deg > 151 && deg < 210)
                    wind_direction = "южный";
                else if (deg > 211 && deg < 240)
                    wind_direction = "юго-западный";
                else if (deg > 241 && deg < 300)
                    wind_direction = "западный";
                else wind_direction = "северный";

                if (temp < 21 && temp > 30)
                    answerOnWether = "Mожешь надеть тапки с носками и ехать на пляжик";
                else if (temp < 20 && temp > 11)
                    answerOnWether = "Сегодня жарко, советую надеть футболку и шортики";
                else if (temp < 10 && temp > 0)
                    answerOnWether = "На улице прохладно, сегодня лучше выйти в джинсах и взять ветровку";
                else if (temp < -1 && temp > -5)
                    answerOnWether = "Лучше надеть пальто, прохладно";
                else if (temp < -6 && temp > -15)
                    answerOnWether = "Уже совсем холодно, пора начинать утепляться и доставать шапку ушанку";
                else if (temp < -16 && temp > -26)
                    answerOnWether = "Пора взять из шкафа теплый пуховик и надеть его";
                else if (temp < -27 && temp > -40)
                    answerOnWether = "На улице такой мороз, оставайся дома и пей чаёк";
                Console.WriteLine(nameOfCity);
                Console.WriteLine(tempOfCity);
                Console.WriteLine(answerOnWether);
                Console.WriteLine(feels_like_temp);
                Console.WriteLine(humidity);
                Console.WriteLine(deg);
                Console.WriteLine(temp);
            }
            catch (WebException)
            {
                Console.WriteLine("Возникло исключение в погоде");
                return;
            }
        }

        public class WeatherResponse
        {
            public TemperatureInfo Main { get; set; }

            public string Name { get; set; }

            public Wind wind { get; set; }
        }
        public class TemperatureInfo
        {
            public float Temp { get; set; }

            public float Feels_like { get; set; }

            public int humidity { get; set; }
        }

        public class Wind
        {
            public float speed { get; set; }

            public int deg { get; set; }
        }


        public class CurrencyRate
        {
            public string CurrencyStringCode;

            public string CurrencyName;

            public double ExchangeRate;
        }
        public static List<CurrencyRate> GetExchangeRates()
        {
            List<CurrencyRate> result = new List<CurrencyRate>();
            XmlSerializer xs = new XmlSerializer(typeof(ValCurs));
            XmlReader xr = new XmlTextReader(@"http://www.cbr.ru/scripts/XML_daily.asp");
            foreach (ValCursValute valute in ((ValCurs)xs.Deserialize(xr)).ValuteList)
            {
                result.Add(new CurrencyRate()
                {
                    CurrencyName = valute.ValuteName,
                    CurrencyStringCode = valute.ValuteStringCode,
                    ExchangeRate = Convert.ToDouble(valute.ExchangeRate)
                });
            }
            Console.WriteLine(result[0]);
            return result;
        }
        public class ValCursValute
        {

            [XmlElementAttribute("CharCode")]
            public string ValuteStringCode;

            [XmlElementAttribute("Name")]
            public string ValuteName;

            [XmlElementAttribute("Value")]
            public string ExchangeRate;
        }
        public class ValCurs
        {
            [XmlElementAttribute("Valute")]
            public ValCursValute[] ValuteList;

        }

        public static void news()
        {
            try
            {
                string url = $"https://newsapi.org/v2/top-headlines?country=ru&apiKey=779e786d05d04c859c25c2922edb4e90";
                HttpWebRequest httpWebRequest = (HttpWebRequest)WebRequest.Create(url);
                HttpWebResponse httpWebResponse = (HttpWebResponse)httpWebRequest.GetResponse();
                string response;

                using (StreamReader streamReader = new StreamReader(httpWebResponse.GetResponseStream()))
                {
                    response = streamReader.ReadToEnd();
                }
                News news = JsonConvert.DeserializeObject<News>(response);

                foreach (Articles articles in news.articles)
                {
                    title.Add(articles.title);
                    ssilka.Add(articles.url);
                    photo.Add(articles.urlToImage);
                    count++;
                }

            }
            catch (WebException)
            {
                Console.WriteLine("Возникло исключение в новостях");
                return;
            }
        }

        public class News
        {
            public int totalResults { get; set; }
            public Articles[] articles { get; set; }
        }

        public class Articles
        {
            public string title { get; set; }
            public string url { get; set; }
            public string urlToImage { get; set; }
        }

        public static void animals_fox()
        {
            try
            {
                string url = $"https://randomfox.ca/floof/";
                HttpWebRequest httpWebRequest = (HttpWebRequest)WebRequest.Create(url);
                HttpWebResponse httpWebResponse = (HttpWebResponse)httpWebRequest.GetResponse();
                string response;

                using (StreamReader streamReader = new StreamReader(httpWebResponse.GetResponseStream()))
                {
                    response = streamReader.ReadToEnd();
                }
                Fox fox = JsonConvert.DeserializeObject<Fox>(response);

                fox_url = fox.image;
            }
            catch (WebException)
            {
                Console.WriteLine("Возникло исключение в новостях");
                return;
            }
        }
        public class Fox
        {
            public string image { get; set; }
        }
        public static void animals_dog()
        {
            try
            {
                string url = $"https://random.dog/woof.json";
                HttpWebRequest httpWebRequest = (HttpWebRequest)WebRequest.Create(url);
                HttpWebResponse httpWebResponse = (HttpWebResponse)httpWebRequest.GetResponse();
                string response;

                using (StreamReader streamReader = new StreamReader(httpWebResponse.GetResponseStream()))
                {
                    response = streamReader.ReadToEnd();
                }
                Dog dog = JsonConvert.DeserializeObject<Dog>(response);

                dog_url = dog.url;
            }
            catch (WebException)
            {
                Console.WriteLine("Возникло исключение в новостях");
                return;
            }
        }
        public class Dog
        {
            public string url { get; set; }
        }
    }
}
