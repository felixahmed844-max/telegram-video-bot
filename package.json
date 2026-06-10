const express = require('express');
const axios = require('axios');
const app = express();

app.use(express.json());

const CONFIG = {
  TELEGRAM_TOKEN: process.env.TELEGRAM_TOKEN || 'ضع_توكن_التليجرام_هنا',
  GEMINI_API_KEY: process.env.GEMINI_API_KEY || 'ضع_مفتاح_جيميني_هنا',
  PEXELS_API_KEY: process.env.PEXELS_API_KEY || 'ضع_مفتاح_بيكسلز_هنا'
};

app.post('/webhook', async (req, res) => {
  try {
    const update = req.body;
    if (!update.message || !update.message.text) return res.sendStatus(200);
    const chatId = update.message.chat.id;
    const messageText = update.message.text;
    const userName = update.message.from.first_name || 'صديقي';
    console.log(`📩 رسالة جديدة من ${userName}: ${messageText}`);
    if (messageText.startsWith('/')) {
      await handleCommand(chatId, messageText, userName);
      return res.sendStatus(200);
    }
    await processVideoRequest(chatId, messageText, userName);
    res.sendStatus(200);
  } catch (error) {
    console.error('❌ خطأ:', error.message);
    res.sendStatus(200);
  }
});

async function processVideoRequest(chatId, topic, userName) {
  try {
    await sendMessage(chatId, `⏳ مرحباً ${userName}!\n\n🎬 جاري العمل على موضوع:\n"${topic}"\n\n⏱️ الوقت المتوقع: دقيقة واحدة...`);
    console.log('📝 كتابة السكريبت...');
    await sendMessage(chatId, '📝 جاري كتابة السكريبت...');
    const script = await generateScript(topic);
    const wordCount = script.split(/\s+/).length;
    console.log('🖼️ جلب الصور...');
    await sendMessage(chatId, '🖼️ جاري جلب الصور من Pexels...');
    const images = await fetchImages(topic);
    console.log('🎨 توليد أفكار Thumbnail...');
    await sendMessage(chatId, '🎨 جاري إنشاء أفكار التصميم...');
    const thumbnailIdeas = await generateThumbnailIdeas(topic);
    const scriptMessage = `✅ *السكريبت جاهز!*\n\n📊 *الإحصائيات:*\n• عدد الكلمات: ${wordCount}\n• الوقت المتوقع: ${Math.ceil(wordCount / 150)} دقائق\n\n━━━━━━━━━━━━━━━━━━\n\n${script}\n\n━━━━━━━━━━━━━━━━━━`;
    await sendMessage(chatId, scriptMessage);
    const imagesMessage = `🖼️ *الصور المقترحة (${images.length} صورة):*\n\n` + images.map((img, i) => `${i + 1}. [تحميل الصورة](${img.url})\n   📸 بواسطة: ${img.photographer}`).join('\n\n');
    await sendMessage(chatId, imagesMessage);
    const thumbnailMessage = `🎨 *أفكار تصميم الـ Thumbnail:*\n\n${thumbnailIdeas}\n\n━━━━━━━━━━━━━━━━━━\n\n💡 *الخطوات التالية:*\n1️⃣ حمّل الصور من الروابط أعلاه\n2️⃣ سجل صوتك وأنت تقرأ السكريبت\n3️⃣ افتح CapCut وابدأ المونتاج\n4️⃣ صمم الـ Thumbnail في Canva\n5️⃣ ارفع على يوتيوب!\n\n🚀 *بالتوفيق يا ${userName}!*`;
    await sendMessage(chatId, thumbnailMessage);
    console.log('✅ تم بنجاح!');
  } catch (error) {
    console.error('❌ خطأ في المعالجة:', error.message);
    await sendMessage(chatId, `❌ عذراً ${userName}، حدث خطأ:\n${error.message}\n\n🔄 حاول مرة أخرى أو تواصل مع المطور.`);
  }
}

async function generateScript(topic) {
  const prompt = `أنت كاتب سكريبتات يوتيوب محترف لقناة "Sky Max" المتخصصة في المحتوى الوثائقي الترفيهي. اكتب سكريبت فيديو وثائقي احترافي عن: ${topic}. المدة: 7-8 دقائق (1200-1500 كلمة). الأسلوب: سردي مشوق + معلومات دقيقة + لمسات فكاهية خفيفة. المقدمة: 15 ثانية قوية جداً. البنية: فقرات واضحة مع عناوين فرعية. الخاتمة: ملخص سريع + دعوة للاشتراك. أضف بين [] إشارات للمونتاج مثل [صورة للحدث] [موسيقى غامضة] [انتقال سريع] [نص متحرك: العنوان].`;
  try {
    const response = await axios.post(`https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=${CONFIG.GEMINI_API_KEY}`, {
      contents: [{ parts: [{ text: prompt }] }],
      generationConfig: { temperature: 0.9, maxOutputTokens: 2048, topP: 0.95 }
    }, { headers: { 'Content-Type': 'application/json' }, timeout: 30000 });
    if (!response.data.candidates || !response.data.candidates[0]) throw new Error('Gemini لم يرجع نتيجة');
    return response.data.candidates[0].content.parts[0].text;
  } catch (error) {
    console.error('❌ خطأ Gemini:', error.message);
    throw new Error('فشل في كتابة السكريبت. تأكد من مفتاح Gemini API.');
  }
}

async function fetchImages(topic) {
  const searchQuery = translateToEnglish(topic);
  try {
    const response = await axios.get(`https://api.pexels.com/v1/search`, {
      params: { query: searchQuery, per_page: 15, orientation: 'landscape' },
      headers: { 'Authorization': CONFIG.PEXELS_API_KEY },
      timeout: 15000
    });
    if (!response.data.photos || response.data.photos.length === 0) throw new Error('لم يتم العثور على صور مناسبة');
    return response.data.photos.map(photo => ({ url: photo.src.large2x, photographer: photo.photographer, alt: photo.alt || searchQuery }));
  } catch (error) {
    console.error('❌ خطأ Pexels:', error.message);
    throw new Error('فشل في جلب الصور. تأكد من مفتاح Pexels API.');
  }
}

async function generateThumbnailIdeas(topic) {
  const prompt = `اقترح 3 أفكار تصميم Thumbnail احترافي وجذاب لفيديو يوتيوب من قناة "Sky Max" عن: ${topic}. لكل فكرة، اذكر العناصر الرئيسية، الألوان، النص (أقل من 5 كلمات بالعربي)، التأثيرات.`;
  try {
    const response = await axios.post(`https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=${CONFIG.GEMINI_API_KEY}`, {
      contents: [{ parts: [{ text: prompt }] }]
    });
    return response.data.candidates[0].content.parts[0].text;
  } catch (error) {
    console.error('❌ خطأ في توليد أفكار Thumbnail:', error.message);
    return 'فشل في توليد الأفكار. استخدم Canva مباشرة.';
  }
}

async function handleCommand(chatId, command, userName) {
  switch (command) {
    case '/start':
      await sendMessage(chatId, `🎬 *مرحباً ${userName} في Sky Max Video Creator!*\n\nأنا بوت ذكي لإنشاء محتوى فيديوهات يوتيوب الوثائقية.\n\n📋 *كيف أستخدمني؟*\nببساطة، أرسل لي موضوع الفيديو وأنا سأجهز لك:\n• 📄 سكريبت كامل (1200-1500 كلمة)\n• 🖼️ 15 صورة عالية الجودة\n• 🎨 أفكار تصميم Thumbnail\n\n💡 *مثال:*\n"أغرب 5 ألغاز في التاريخ"\n\n🚀 *جربني الآن!*`);
      break;
    case '/help':
      await sendMessage(chatId, `❓ *المساعدة*\n/start - البدء\n/help - المساعدة\n/about - عن البوت\n\nأرسل موضوع الفيديو مباشرة.`);
      break;
    case '/about':
      await sendMessage(chatId, `ℹ️ *عن البوت*\nنسخة 1.0.0 | القناة: Sky Max | Gemini AI | Pexels API`);
      break;
    default:
      await sendMessage(chatId, `❓ أمر غير معروف.\nاستخدم /help`);
  }
}

async function sendMessage(chatId, text) {
  try {
    await axios.post(`https://api.telegram.org/bot${CONFIG.TELEGRAM_TOKEN}/sendMessage`, {
      chat_id: chatId, text: text, parse_mode: 'Markdown', disable_web_page_preview: true
    }, { timeout: 10000 });
  } catch (error) {
    console.error('❌ خطأ في إرسال الرسالة:', error.message);
  }
}

function translateToEnglish(arabicText) {
  const translations = { 'ألغاز': 'mysteries', 'تاريخ': 'history', 'غامض': 'mysterious', 'عجائب': 'wonders', 'حقائق': 'facts', 'أسرار': 'secrets', 'قصص': 'stories', 'حرب': 'war', 'علم': 'science', 'فضاء': 'space', 'طبيعة': 'nature', 'حيوانات': 'animals' };
  let result = arabicText.toLowerCase();
  for (const [arabic, english] of Object.entries(translations)) result = result.replace(new RegExp(arabic, 'g'), english);
  return result || 'documentary';
}

app.get('/', (req, res) => {
  res.send(`<!DOCTYPE html><html dir="rtl"><head><meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1.0"><title>Sky Max Bot</title><style>body{font-family:sans-serif;background:linear-gradient(135deg,#667eea,#764ba2);color:white;text-align:center;padding:50px}.container{background:rgba(255,255,255,0.1);padding:40px;border-radius:20px}.status{background:#4caf50;display:inline-block;padding:10px 20px;border-radius:25px}</style></head><body><div class='container'><h1>🎬 Sky Max Video Creator</h1><div class='status'>✅ البوت يعمل بنجاح!</div><p>Telegram: @${CONFIG.TELEGRAM_TOKEN ? 'your_bot_username' : 'لم يتم إعداد التوكن بعد'}</p></div></body></html>`);
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`✅ السيرفر يعمل على المنفذ ${PORT}`));

process.on('unhandledRejection', (error) => console.error('❌ خطأ غير معالج:', error));
