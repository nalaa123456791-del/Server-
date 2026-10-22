const express = require('express');
const cors = require('cors');
const nodemailer = require('nodemailer');
const admin = require('firebase-admin');

// --- Firebase Admin Initialization (صرف یوزرز اور چیٹ ہسٹری کے لیے) ---
const firebaseConfig = {
  apiKey: "AIzaSyC_9bLIJlt5a4TuiAgiJh3HvZ940b4r7lY",
  authDomain: "aab-ai.firebaseapp.com",
  databaseURL: "https://aab-ai-default-rtdb.firebaseio.com",
  projectId: "aab-ai",
  storageBucket: "aab-ai.firebasestorage.app",
  messagingSenderId: "464117954095",
  appId: "1:464117954095:web:71fb7abeb006e5b570f8da"
};

if (!admin.apps.length) {
  admin.initializeApp({
    credential: admin.credential.cert({
      projectId: firebaseConfig.projectId,
      clientEmail: "firebase-adminsdk@aab-ai.iam.gserviceaccount.com",
      privateKey: (process.env.FIREBASE_PRIVATE_KEY || "").replace(/\\n/g, '\n')
    }),
    databaseURL: firebaseConfig.databaseURL
  });
}
const db = admin.database();

const app = express();
app.use(express.json({ limit: '50mb' }));
app.use(cors());

// =========================================================================
// 🔑 AAB AI - CONFIGURATION & CREDENTIALS
// =========================================================================
const APP_NAME = "AAB AI";
const GMAIL_USER = 'Nalaa123456791@gmail.com'; 
const GMAIL_APP_PASSWORD = 'dmhnxyytkvmgjqov'; // آپ کا Gmail App Password
const SERVER_API_KEY = "AAbai/chat-bottl/phph-ca45-hygcs-8ijnhytfc/lkjn-2026"; 
// =========================================================================

// 📧 ای میل الرٹ ٹرانسپورٹر
const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: GMAIL_USER,
    pass: GMAIL_APP_PASSWORD
  }
});

// =========================================================================
// 🧠 PRODUCTION KNOWLEDGE BASE (آپ کے تمام سوالات اور جوابات)
// =========================================================================
const PRODUCTION_KNOWLEDGE_BASE = [
  // --- AI aur Buniyadi Maloomat ---
  { keywords: ["aap kaun hain", "kaun ho", "who are you", "کون ہیں"], answer: "Main Claude hoon, Anthropic ka banaya hua AI assistant." },
  { keywords: ["ai kya hota hai", "artificial intelligence", "اے آئی kya hai"], answer: "AI (Artificial Intelligence) aisi technology hai jo machines ko insaani soch, seekhne aur faislay karne ke qabil banati hai." },
  { keywords: ["kya aap insaan hain", "human", "کیا آپ انسان"], answer: "Nahi, main ek AI program hoon." },
  { keywords: ["kaise kaam karte hain", "kaam", "کام کیسے"], answer: "Mujhe bohot saare text data par train kiya gaya hai, jis se main sawal samajh kar jawab likhta hoon." },
  { keywords: ["galti kar sakte hain", "mistake", "غلطی"], answer: "Ji haan, isliye zaroori maloomat dobara check kar lena behtar hai." },
  { keywords: ["baatein yaad rakhte hain", "memory", "یاد"], answer: "Aam tor par ek conversation ke andar yaad rakhta hoon. Memory feature on ho to usi ke mutabiq." },
  { keywords: ["internet use", "browse", "انٹرنیٹ"], answer: "Jab web search on ho to haan, current maloomat dhoond sakta hoon." },
  { keywords: ["urdu samajhte hain", "اردو"], answer: "Ji haan, Urdu, Roman Urdu aur kai doosri zubanein samajhta hoon." },
  { keywords: ["chatgpt aur claude", "farq", "فرق"], answer: "Dono alag companies ke AI assistants hain. Har ek ki apni training, khasiyat aur andaz hai. Behtar yeh hai ke dono try kar ke dekhein." },
  { keywords: ["job le lega", "nokri", "نوکری"], answer: "AI kuch kaam khud kar sakta hai, lekin naye kaam bhi paida karta hai. Jo log AI istemal karna seekhte hain woh aage rehte hain." },
  { keywords: ["prompt kya hota hai", "prompt"], answer: "Woh sawal ya hidayat jo aap AI ko dete hain." },
  { keywords: ["achha prompt", "behtar prompt"], answer: "Saaf likhein ke kya chahiye, context dein, aur format batayein (jaise “5 points mein”)." },
  { keywords: ["bharosa", "trust"], answer: "Aam kaamon mein madadgar hai, lekin sehat, qanoon aur paise ke faislon mein ahem maloomat khud verify karein." },
  { keywords: ["private", "rahti hain", "پرائیویٹ"], answer: "Privacy policy platform par depend karti hai. Apne password, CNIC ya bank details kabhi share na karein." },
  { keywords: ["tasveer bana sakta hai", "image", "تصویر"], answer: "Kuch AI tools bana sakte hain. Main khud SVG, charts aur diagrams bana sakta hoon." },
  { keywords: ["file parh sakte hain", "file", "فائل"], answer: "Ji, PDF, Word, Excel aur images upload karein to main parh kar madad kar sakta hoon." },
  { keywords: ["translate", "ترجمہ"], answer: "Ji, Urdu, English, Arabic aur kai zubanon mein." },
  { keywords: ["kisne banaya", "creator"], answer: "Mujhe Anthropic ne banaya hai." },

  // --- Likhna aur Content ---
  { keywords: ["email kaise likhun", "email"], answer: "Subject, salam, mukhtasar matlab, aur shukriya, in 4 cheezon ko shamil karein. Batayein kis ko aur kya likhna hai, main draft bana dunga." },
  { keywords: ["cv", "resume", "سی وی"], answer: "Naam, rabta, summary, tajurba, taleem aur skills likhein. Ek page rakhein aur nokri ke mutabiq customize karein." },
  { keywords: ["cover letter"], answer: "3 paragraph: apna taaruf, aap company ke liye kyun behtar hain, aur akhir mein shukriya." },
  { keywords: ["essay", "مضمون"], answer: "Introduction, body paragraphs aur conclusion." },
  { keywords: ["content ko behtar", "improve content"], answer: "Chhote jumlay, saaf maqsad, aur examples shamil karein." },
  { keywords: ["poetry", "shayari", "شاعری"], answer: "Ji, mauzu aur mood bata dein (udaasi, mohabbat, watan)." },

  // --- Parhai aur Imtihan ---
  { keywords: ["imtihan", "exam", "امتحان"], answer: "Syllabus ko hisson mein baantein, rozana timetable banayein, practice papers hal karein." },
  { keywords: ["yaad kaise karun", "remember", "یاد"], answer: "Active recall, repetition aur khud ko test karna sab se kargar tareeqay hain." },
  { keywords: ["concentration", "دھیان"], answer: "25 minute parhain, 5 minute break lein (Pomodoro), aur phone door rakhein." },
  { keywords: ["scholarship", "स्कॉलरशिप"], answer: "Achhay marks, mazboot application, aur deadline se pehle apply karna zaroori hai." },
  { keywords: ["ielts"], answer: "Rozana listening, reading, writing aur speaking ki practice karein aur purane papers hal karein." },
  { keywords: ["research paper"], answer: "Topic chunein, literature review karein, methodology likhein, results aur conclusion dein." },

  // --- Coding aur Technology ---
  { keywords: ["programming kahan se", "coding", "کوڈنگ"], answer: "Python se shuru karein. Free resources: freeCodeCamp, YouTube, CS50." },
  { keywords: ["python kya hai", "python"], answer: "Aasaan aur mashhoor programming language jo AI, web aur data science mein use hoti hai." },
  { keywords: ["website kaise banayen", "website"], answer: "HTML, CSS, JavaScript seekhein, ya WordPress/Wix jaisay tools use karein." },
  { keywords: ["app kaise banayen", "app", "ایپ"], answer: "Flutter ya React Native se ek code se Android aur iOS dono ke liye app ban sakti hai." },
  { keywords: ["error de raha hai", "bug", "ایرور"], answer: "Error message aur code mujhe paste kar dein, main wajah aur hal bata dunga." },
  { keywords: ["hacking seekhni hai", "hacking", "ہیکنگ"], answer: "Ethical hacking/cybersecurity ke courses (jaise CompTIA Security+, TryHackMe) se qanooni tareeqay se shuru karein." },
  { keywords: ["computer slow", "سلو"], answer: "Faltu programs band karein, storage saaf karein, antivirus scan chalayein, ya RAM/SSD upgrade karein." },

  // --- Career aur Paisa ---
  { keywords: ["online paisa", "paisa kamayen", "پیسہ"], answer: "Freelancing (Fiverr, Upwork), content creation, tutoring ya e-commerce." },
  { keywords: ["freelancing", "فری لانسنگ"], answer: "Ek skill chunein, portfolio banayein, profile bana kar chhote projects se shuru karein." },
  { keywords: ["interview", "انٹرویو"], answer: "Company ke baare mein parhain, aam sawalat ki practice karein, aur apne tajurbay ki misalein tayyar rakhein." },
  { keywords: ["business idea", "بزنس"], answer: "Apni skills aur local zaroorat dekhein, jaise online store, digital services, ya food business." },
  { keywords: ["bachat", "savings", "بچت"], answer: "Mahana budget banayein, aamdani ka kam az kam 20% pehle alag rakhein." },

  // --- Sehat aur Zindagi ---
  { keywords: ["wazan", "weight", "وزن"], answer: "Mutavazin khuraak, rozana chalna ya warzish, aur bharpoor neend. (Bimari ho to doctor se mashwara karein.)" },
  { keywords: ["neend nahi aati", "sleep", "نیند"], answer: "Sone ka waqt muqarrar karein, sone se pehle screen se bachein, aur chai/coffee kam karein." },
  { keywords: ["stress", "tension", "ٹینشن"], answer: "Gehri saansein, chehal qadmi, doston se baat, aur zaroorat ho to mahir se rabta karein." },
  { keywords: ["sar dard", "headache", "سر درد"], answer: "Aaram, paani aur neend. Baar baar ya shadeed dard ho to doctor ko dikhayein." },

  // --- Rozmarra aur Safar ---
  { keywords: ["visa", "ویسا"], answer: "Mulk ke embassy/official website par requirements dekhein: passport, bank statement, invitation/admission letter, waghera." },
  { keywords: ["mausam", "weather", "موسم"], answer: "Apna shehar bata dein, main check kar dunga." },
  { keywords: ["kitab", "books", "کتاب"], answer: "Apni pasand (fiction, self-help, tareekh) bata dein to main behtar suggestions dunga." },
  { keywords: ["boredom", "boring", "بوریت"], answer: "Koi nayi skill seekhein, kitab parhein, walk par jayein, ya main ke saath koi game/quiz khel lein." },

  // --- Zubaan aur Taleem (Mazeed) ---
  { keywords: ["english bolna", "spoken english"], answer: "Rozana bolne ki practice karein, English films/podcasts sunein, aur galtiyon se na gabrayein." },
  { keywords: ["vocabulary", "alfaz"], answer: "Rozana 5 naye alfaz seekhein aur unhein jumlon mein istemal karein." },
  { keywords: ["grammar"], answer: "Basic tenses se shuru karein, phir practice exercises karein." },
  { keywords: ["arabic", "عربی"], answer: "Pehle haroof aur basic alfaz se shuru karein, phir chhote jumlay seekhein." },
  { keywords: ["matric", "inter", "کیپس"], answer: "Apni dilchaspi, marks aur career scope dekh kar field chunein (engineering, medical, IT, business waghera)." },
  { keywords: ["university admission", "admission"], answer: "Entry test (NTS/ECAT/MDCAT waghera), merit aur deadlines dekhein aur documents tayyar rakhein." },
  { keywords: ["online courses", "کورసেস"], answer: "Coursera, edX, Udemy, Khan Academy aur YouTube." },
  { keywords: ["mathematics", "math", "ریاضی"], answer: "Basics se shuru karein, rozana questions hal karein, aur har ghalti se seekhein." },
  { keywords: ["notes kaise banayen", "notes"], answer: "Mukhtasar points likhein, headings aur colors istemal karein, aur baad mein revise karein." },

  // --- Business aur Marketing (Mazeed) ---
  { keywords: ["online store", "shopify"], answer: "Product chunein, Shopify/Daraz/WooCommerce jaisa platform lein, payment aur delivery ka intizam karein." },
  { keywords: ["social media marketing"], answer: "Apne customers ki pehchan karein, regular content daalein, aur engagement dekh kar strategy badlein." },
  { keywords: ["logo kaise", "logo"], answer: "Canva ya Figma istemal karein, ya designer se banwayein. Saada aur yaadgar rakhein." },
  { keywords: ["seo kya hai", "seo"], answer: "Website ko Google search mein upar laane ka amal." },
  { keywords: ["business plan"], answer: "Idea, market research, maali plan, marketing aur risks shamil karein." },
  { keywords: ["customers kaise", "customers"], answer: "Achhi service, positive reviews, referral aur targeted ads." },
  { keywords: ["startup ke liye paisa"], answer: "Apni bachat, dost/family, investors, ya government/NGO ke programs." },
  { keywords: ["youtube channel", "يوتيوب"], answer: "Mauzu chunein, saaf awaaz/video banayein, regular upload karein, aur titles/thumbnails par mehnat karein." },
  { keywords: ["dropshipping"], answer: "Aisa business jahan aap saaman stock nahi karte, supplier seedha customer ko bhejta hai." },
  { keywords: ["invoice"], answer: "Apna aur customer ka naam, tareekh, items, qeemat, tax aur total likhein. Excel ya online tools kaam aate hain." },

  // --- Sehat aur Khuraak (Mazeed) ---
  { keywords: ["pani kitna", "water"], answer: "Aam tor par rozana 6 se 8 gilaas, mausam aur kaam ke hisaab se zyada ho sakta hai." },
  { keywords: ["nashta", "breakfast"], answer: "Zyadatar logon ke liye faidamand hai, lekin zaroori nahi. Apne jism ki zaroorat dekhein." },
  { keywords: ["bal jhadne", "hair fall"], answer: "Mutavazin khuraak, iron aur protein, tension kam karein. Zyada ho to doctor ko dikhayein." },
  { keywords: ["daane", "pimples", "acne"], answer: "Hormones, tail wali jild, gandagi ya khuraak. Chehra dhoyein aur zyada ho to dermatologist se milein." },
  { keywords: ["gym", "workout"], answer: "Halki warzish se shuru karein, sahi form seekhein, aur dheere dheere barhayein." },
  { keywords: ["kamar dard", "back pain"], answer: "Sahi posture, halka stretching aur aaram. Dard mustaqil ho to doctor se rujoo karein." },
  { keywords: ["sehatmand khuraak", "diet"], answer: "Sabzi, phal, daal, protein aur kam cheeni/tali hui cheezein." },
  { keywords: ["roza", "fasting"], answer: "Kai logon ko nizam aur sehat mein faida hota hai, lekin bimar log doctor se mashwara karein." },
  { keywords: ["dimaghi sukoon", "mental peace"], answer: "Ibadat, warzish, achhi neend, doston se baat, aur zaroorat par mahir se madad." },
  { keywords: ["chhoti si bimari", "fever"], answer: "Aaram aur paani lein. Bukhar ya dard barhay ya kai din rahe to doctor ko dikhayein." },

  // --- Technology aur Security (Mazeed) ---
  { keywords: ["battery", "بٹری"], answer: "Screen brightness kam karein, background apps band karein, aur battery health check karein. Purani battery ho to badalwa lein." },
  { keywords: ["phone hang", "hang"], answer: "Faltu apps hata dein, storage khali karein, phone restart karein, ya zaroorat par factory reset karein (pehle backup lein)." },
  { keywords: ["wifi slow", "وائی فائی"], answer: "Router restart karein, usay khuli jagah rakhein, aur dekhein ke kitne devices juray hain." },
  { keywords: ["password mazboot", "password"], answer: "Kam az kam 12 haroof, alfaz, number aur symbols milayein. Har account ka alag password rakhein." },
  { keywords: ["account hack", "hacked"], answer: "Two-factor authentication on karein, shak wale links par click na karein, aur password manager istemal karein." },
  { keywords: ["phishing"], answer: "Jaali email ya message jo aap se password ya bank details nikalwane ki koshish karta hai." },
  { keywords: ["data backup", "backup"], answer: "Google Drive, iCloud ya external hard drive par regular copy rakhein." },
  { keywords: ["vpn kya hai", "vpn"], answer: "Aisi service jo aap ka internet traffic encrypt kar ke aap ki location chupa deti hai. Qanooni hudood ka khayal rakhein." },
  { keywords: ["fake news", "افواہ"], answer: "Source dekhein, doosri mo’atabar websites par check karein, aur sansanikhez headlines se hoshiyar rahein." },
  { keywords: ["cloud storage"], answer: "Internet par data rakhne ki jagah, jahan se aap kisi bhi device se file khol saktay hain." },

  // --- AI aur Technology (Mazeed Advanced) ---
  { keywords: ["machine learning", "ml"], answer: "AI ka woh hissa jis mein computer data se khud seekhta hai, bina har cheez ki programming ke." },
  { keywords: ["assignment", "homework"], answer: "Madad lena (samajhna, structure banana) theek hai, lekin poora kaam copy karna institute ke qawaid ke khilaf ho sakta hai aur seekhne ka nuqsan hai." },
  { keywords: ["tasveer kaise banwayen", "generate image"], answer: "Image generation tools mein detail ke saath tafseel likhein, jaise rang, andaaz aur manzar." },
  { keywords: ["chatbot kaise banayen", "chatbot"], answer: "API (jaise Claude ya OpenAI) istemal karein, ya no-code tools se shuru karein." },
  { keywords: ["data science"], answer: "Data se maloomat nikalne ka fan, jis mein statistics, programming aur visualization shamil hain." },
  { keywords: ["blockchain"], answer: "Aisi digital bahi jo kayi computers par mehfooz hoti hai aur badli nahi ja sakti." },
  { keywords: ["cryptocurrency", "bitcoin"], answer: "Yeh bohot risky hai aur qeemat tezi se oopar neeche hoti hai. Sirf utna lagayein jitna kho dene ka bojh na ho. Main financial advisor nahi hoon, mahir se mashwara lein." },
  { keywords: ["excel", "formulas"], answer: "SUM, IF, VLOOKUP/XLOOKUP se shuru karein aur rozana practice karein." },
  { keywords: ["video edit", "capcut"], answer: "CapCut ya DaVinci Resolve se shuru karein, aur YouTube tutorials dekhein." },
  { keywords: ["laptop", "لابتاپ"], answer: "Budget, kaam (parhai, gaming, design) aur RAM/SSD dekh kar chunein. Batayein to main options suggest karun." },

  // --- Rishtay aur Rozmarra Zindagi ---
  { keywords: ["dosti", "friends", "دوستی"], answer: "Dilchaspi ke groups join karein, khul kar baat karein, aur doosron mein sachi dilchaspi lein." },
  { keywords: ["ghar walon se jhagra", "family"], answer: "Thanda hone ka intezaار karein, sukoon se apni baat rakhein, aur doosray ka nuqta bhi suneen." },
  { keywords: ["sharmeela pan", "shyness"], answer: "Chhoti baatcheet se shuru karein, rozana khud ko thora challenge karein, aur practice karte rahein." },
  { keywords: ["aitemad", "confidence"], answer: "Chhote maqsad rakhein, kamyabiyon ko yaad rakhein, aur apna muqabla doosron se na karein." },
  { keywords: ["time management", "وقت"], answer: "Rozana ki fehrist banayein, aham kaam pehle karein, aur faltu cheezon (phone/social media) ka waqt tay karein." },
  { keywords: ["procrastination", "taalmatol"], answer: "Kaam ko chhote hissay mein baantein aur sirf 5 minute se shuru kar dein." },
  { keywords: ["aadatein", "habits"], answer: "Ek waqt mein ek aadat chunein, usay rozana ek hi waqt par karein, aur 21 se 66 din tak jari rakhein." },
  { keywords: ["gharelu budget", "budget"], answer: "Aamdani likhein, zaroori kharche alag karein, bachat tay karein, aur har mahine jaiza lein." },
  { keywords: ["shadi ke liye rishta", "rishta"], answer: "Khandan, dost aur mo’atabar rishta services se madad lein, aur mizaj, ikhlaaq aur maqasid ko ahmiyat dein." },
  { keywords: ["bachon ki tarbiyat", "parenting"], answer: "Pyar, misal bankar sikhana, waqt dena, aur hadd mein rehte hue azadi dena." },

  // --- Pakistan aur Aam Maloomat ---
  { keywords: ["cnic", "nadra", "شناختی کارڈ"], answer: "NADRA office jayein ya Pak ID app/website se online apply karein. Zaroori documents saath rakhein." },
  { keywords: ["passport", "پاسپورٹ"], answer: "Online passport portal par form bharein, fees jama karein aur appointment par documents ke saath jayein." },
  { keywords: ["driving license", "لائسنس"], answer: "Learner permit lein, test pass karein aur licensing office se license banwayein. Requirements shehar ke hisaab se alag ho sakti hain." },
  { keywords: ["bijli ka bill", "bill"], answer: "Apni distribution company ki website par ya reference number se online check kar sakte hain." },
  { keywords: ["tax return", "fbr"], answer: "FBR ke IRIS portal par registration karein aur apni aamdani ke mutabiq return jama karein. Pechida ho to tax consultant se mashwara lein." },
  { keywords: ["pakistan mein shehar", "tourist"], answer: "Hunza, Swat, Naran, Skardu, Murree, Lahore aur Islamabad mashhoor hain." },
  { keywords: ["bank account", "bank"], answer: "CNIC, tasveer aur initial deposit ke saath bank ki branch ya app se apply karein. Roshan Digital Account bhi ek option hai." },
  { keywords: ["easypaisa", "jazzcash"], answer: "App download karein, CNIC se register karein, aur paise bhejne, bill ada karne waghera ke liye istemal karein." },
  { keywords: ["government job", "jobs"], answer: "PPSC, FPSC aur NTS ke ads dekhein, test ki tayyari karein, aur documents tayyar rakhein." },
  { keywords: ["qanooni masla", "lawyer"], answer: "Mustanad wakeel se mashwara karein. Main aam maloomat de sakta hoon, lekin main wakeel nahi hoon." },

  // --- Programming aur Coding Basics ---
  { keywords: ["html aur css", "html"], answer: "HTML website ka dhancha (structure) banati hai, jabke CSS usay rang, size aur design deti hai." },
  { keywords: ["javascript kya hai", "javascript"], answer: "Aisi language jo website ko interactive banati hai, jaise buttons, forms aur animations." },
  { keywords: ["hello world", "python print"], answer: `print("Hello World") likhein aur run karein.` },
  { keywords: ["api kya hota hai", "api"], answer: "Aisa zariya jis se do software aapas mein baat karte hain aur data share karte hain." },
  { keywords: ["database kya hai", "database"], answer: "Data ko tarteeb se mehfooz karne ki jagah, jaise MySQL, PostgreSQL ya MongoDB." },
  { keywords: ["git aur github", "git"], answer: "Git code ke versions track karta hai, GitHub us code ko online rakhne aur share karne ka platform hai." },
  { keywords: ["frontend aur backend", "frontend"], answer: "Frontend woh hai jo user dekhta hai, backend woh hai jo parde ke peeche server aur data sambhalta hai." },
  { keywords: ["bug kya hota hai", "bug"], answer: "Code mein ghalti jis ki wajah se program ghalat chalta hai ya crash ho jata hai." },
  { keywords: ["coding mein kamzor", "learn coding"], answer: "Chhote projects banayein, rozana practice karein, aur masle khud hal karne ki koshish karein." },
  { keywords: ["language seekhun", "best language"], answer: "Shuruaat ke liye Python, web ke liye JavaScript, aur Android ke liye Kotlin ya Flutter (Dart)." },
  { keywords: ["web development career", "web dev"], answer: "Demand achhi hai. Portfolio aur projects banayein, phir freelancing ya job dono mumkin hain." },
  { keywords: ["open source"], answer: "Aisa software jis ka code sab ke liye khula hota hai, istemal aur behtar karne ke liye." },
  { keywords: ["algorithm kya hota hai", "algorithm"], answer: "Kisi masle ko hal karne ke qadamba qadam tareeqe ko algorithm kehte hain." },
  { keywords: ["variable kya hota hai", "variable"], answer: "Data ko naam de kar store karne wali jagah, jaise age = 20." },
  { keywords: ["loop kya hota hai", "loop"], answer: "Ek kaam ko baar baar karwane ka tareeqa, jaise for aur while." },

  // --- Parhai aur Career (Mazeed) ---
  { keywords: ["scope zyada hai", "best field"], answer: "IT, AI, data science, medical, engineering aur digital marketing mein scope hai. Apni dilchaspi bhi dekhein." },
  { keywords: ["abroad parhai", "study abroad"], answer: "Country aur university chunein, IELTS/GRE jaisay tests dein, scholarship dhoondein aur documents tayyar rakhein." },
  { keywords: ["linkedin profile", "linkedin"], answer: "Saaf tasveer, headline, summary, tajurba aur skills likhein, aur apne field ke logon se connect karein." },
  { keywords: ["apne baare mein batayein", "tell me about yourself"], answer: "1 minute mein apna naam, taleem, tajurba aur apni khasiyat batayein, aur is job se ta’alluq jorein." },
  { keywords: ["salary barhwane", "salary"], answer: "Apni kamyabiyon ki list rakhein, market rate dekhein, aur munasib waqt par dalil ke saath baat karein." },
  { keywords: ["kam umri mein kamyab", "success"], answer: "Seekhte rahein, mehnat karein, achhe log chunein, aur sabr rakhein." },
  { keywords: ["presentation", "پریزنٹیشن"], answer: "Saaf slides banayein, kam text rakhein, pehle se practice karein, aur logon se nazar mila kar baat karein." },
  { keywords: ["public speaking", "speaking"], answer: "Chhote group se shuru karein, tayyari karein, aur gehri saans le kar baat karein." },
  { keywords: ["team mein kaam", "teamwork"], answer: "Saaf baat karein, doosron ki raaye sunein, aur zimmedari mil kar baantein." },
  { keywords: ["burnout", "تھکن"], answer: "Waqfa lein, neend poori karein, kaam ki hadd tay karein, aur shauq ke liye waqt nikalein." },
  { keywords: ["notes revise", "revision"], answer: "Mind maps, flashcards aur mukhtasar summary banayein." },
  { keywords: ["group study"], answer: "Ji, agar sab tayyar ho kar aayein aur dhyan parhai par rahe to sawal-jawab se bohot madad milti hai." },
  { keywords: ["scholarship essay"], answer: "Apni kahani sachchai se likhein, maqsad aur mushkilat batayein, aur batayein ke scholarship se kya badlega." },
  { keywords: ["internship", "انٹرنشپ"], answer: "LinkedIn, Rozee.pk, university career office aur seedha companies ko email karke." },
  { keywords: ["skills ya degree", "degree vs skills"], answer: "Dono ahem hain. Degree buniyad deti hai aur skills nokri aur kamai mein madad karti hain." },

  // --- Sehat aur Zindagi (Mazeed) ---
  { keywords: ["neend kitni", "hours of sleep"], answer: "Baligh logon ko aam tor par 7 se 9 ghante chahiye." },
  { keywords: ["vitamin d ki kami", "vitamin d"], answer: "Dhoop, khuraak (anda, machli) aur zaroorat par doctor ke mashware se supplements." },
  { keywords: ["aankhon ki hifazat", "eyes"], answer: "Screen se waqfa lein (20-20-20 rule), roshni theek rakhein aur check-up karwayein." },
  { keywords: ["chai coffee zyada", "tea coffee"], answer: "Had se zyada neend, ghabrahat aur acidity paida kar sakti hai. Din mein 2 se 3 cup tak theek mana jata hai." },
  { keywords: ["acidity ka ilaj", "acidity"], answer: "Halka aur kam masalay wala khana, ek saath zyada na khayein, aur sote waqt khana na khayein. Barqarar ho to doctor ko dikhayein." },
  { keywords: ["sardi zukam", "cold flu"], answer: "Garam pani, aaram, vitamin C wali cheezein. Bukhar ya takleef barhay ya kai din rahe to doctor ko milein." },
  { keywords: ["walking ke fawaid", "walk"], answer: "Dil, wazan, mood aur neend sab behtar hote hain. Rozana 30 minute ka maqsad rakhein." },
  { keywords: ["dil ki sehat", "heart health"], answer: "Tali cheezein kam karein, warzish karein, namak aur cheeni mehdood rakhein, aur sigarette se bachein." },
  { keywords: ["diabetes se bachun", "diabetes"], answer: "Wazan qaabu mein rakhein, meethi cheezein kam karein, warzish karein aur waqt par check-up karwayein." },
  { keywords: ["depression ke ilaj", "depression"], answer: "Agar udaasi aur bechaini mustaqil ho to maahir nafsiyat ya doctor se rujoo karein. Yeh ilaaj se theek ho sakti hai." }
];

// --- API Key سیکیورٹی مڈل ویئر ---
function authenticateApiKey(req, res, next) {
  const userApiKey = req.headers['x-api-key'];
  if (!userApiKey || userApiKey !== SERVER_API_KEY) {
    return res.status(403).json({ success: false, message: "دسترسی ممنوع! غلط API Key۔" });
  }
  next();
}

// 🚨 سیکیورٹی الرٹ ای میل بھیجنے کا فنکشن
async function sendSecurityAlertEmail(suspectQuestion, refCode) {
  try {
    const serverDomain = process.env.SERVER_URL || "http://localhost:3000";
    const blockLink = `${serverDomain}/api/admin/block-user?refCode=${encodeURIComponent(refCode || '')}&secretKey=${SERVER_API_KEY}`;

    const mailOptions = {
      from: GMAIL_USER,
      to: GMAIL_USER,
      subject: `🚨 [SECURITY ALERT] AAB AI - غیر قانونی سرگرمی کا پتہ چلا!`,
      text: `خبردار!\n\nکسی صارف نے سیکیورٹی پالیسی کے خلاف ورزی کی کوشش کی ہے:\n\n- پوچھا گیا سوال: "${suspectQuestion}"\n- ریفرنس کوڈ: ${refCode || "غیر موجود"}\n- وقت: ${new Date().toLocaleString()}\n\nاس صارف یا چینل کو فائر بیس میں بلاک کرنے کے لیے نیچے دیے گئے لنک پر کلک کریں:\n${blockLink}`
    };

    await transporter.sendMail(mailOptions);
    console.log("🚨 سیکیورٹی الرٹ ای میل کامیابی سے Nalaa123456791@gmail.com پر بھیج دی گئی ہے!");
  } catch (error) {
    console.error("ای میل بھیجنے میں خرابی:", error);
  }
}

// ⛔ ایڈمن بلاکنگ API
app.get('/api/admin/block-user', async (req, res) => {
  const { refCode, secretKey } = req.query;

  if (secretKey !== SERVER_API_KEY) {
    return res.status(403).send("<h1>403 Forbidden - غير مجاز رسائی</h1>");
  }

  if (!refCode) {
    return res.status(400).send("<h1>400 Bad Request - یوزر ریفرنس کوڈ غائب ہے</h1>");
  }

  try {
    await db.ref('users/' + refCode).update({ isBlocked: true });
    res.send(`<h1>✅ صارف / چینل (${refCode}) کو فائر بیس میں کامیابی سے بلاک کر دیا گیا ہے!</h1>`);
  } catch (error) {
    res.status(500).send("<h1>فائر بیس ڈیٹا بیس ایرر</h1>");
  }
});

// --- Main AI Processing & Security API ---
app.post('/api/ask', authenticateApiKey, async (req, res) => {
  const { question, refCode, hasImage } = req.body;
  
  // 1. خطرناک یا غیر قانونی الفاظ کا فلٹر (زہر، ہیکنگ وغیرہ)
  const unsafeKeywords = ["زہر", "zeher", "poison", "kill", "suicide", "بم", "bomb", "hack", "ہیک"];
  const lowerQ = (question || "").toLowerCase();
  const isDangerous = unsafeKeywords.some(keyword => lowerQ.includes(keyword));

  if (isDangerous) {
    sendSecurityAlertEmail(question, refCode);

    return res.status(403).json({
      success: false,
      appName: APP_NAME,
      isWarning: true,
      answer: "⚠️ [SECURITY WARNING]: معذرت! آپ کا یہ سوال ہماری سیکیورٹی پالیسی کے خلاف ہے۔ ہیکنگ یا نقصان دہ معلومات کی اجازت نہیں ہے۔"
    });
  }

  // 2. فائر بیس سے چیک کریں کہ کیا یہ صارف بلاک ہے
  const userRef = db.ref('users/' + (refCode || 'guest'));
  const snapshot = await userRef.once('value');
  const userData = snapshot.val() || {};

  if (userData.isBlocked) {
    return res.status(403).json({ 
      success: false, 
      message: "آپ کا اکاؤنٹ یا چینل بلاک کر دیا گیا ہے۔" 
    });
  }

  // 3. AI تصویر بنانے کا انجن (انسانوں یا کسی بھی چیز کی تصویر کے لیے)
  if (
    hasImage || 
    lowerQ.includes("photo") || 
    lowerQ.includes("image") || 
    lowerQ.includes("تصویر") || 
    lowerQ.includes("بناؤ") || 
    lowerQ.includes("بنا دو") ||
    lowerQ.includes("girl") ||
    lowerQ.includes("boy") ||
    lowerQ.includes("aurat")
  ) {
    let constructedPrompt = `${question}, photorealistic 8k render, hyper-detailed`;
    const encodedPrompt = encodeURIComponent(constructedPrompt);
    const generatedImageUrl = `https://image.pollinations.ai/prompt/${encodedPrompt}?width=1024&height=1024&nologo=true&seed=${Math.floor(Math.random() * 1000000)}`;

    // چیٹ اور تصویر کا ریکارڈ فائر بیس میں سیو کریں
    await db.ref('chats/').push({
      refCode: refCode || 'guest',
      question: question,
      imageUrl: generatedImageUrl,
      timestamp: Date.now()
    });

    return res.json({
      success: true,
      appName: APP_NAME,
      isImage: true,
      imageUrl: generatedImageUrl,
      answer: `آپ کی مانگی گئی تصویر تیار ہے! 👇`
    });
  }

  // 4. سرور کے اپنے Knowledge Base سے جواب تلاش کرنا
  let matchedAnswer = null;
  for (let item of PRODUCTION_KNOWLEDGE_BASE) {
    let matchCount = item.keywords.filter(kw => lowerQ.includes(kw) || isSimilar(lowerQ, kw)).length;
    if (matchCount > 0) {
      matchedAnswer = item.answer;
      break;
    }
  }

  let finalAnswer = matchedAnswer || `آپ کے سوال کا تجزیہ کر لیا گیا ہے۔ AAB AI پورٹل پر اس بارے میں معلومات درج ہیں۔`;

  // سوال اور عام جواب بھی فائر بیس میں ہسٹری کے لیے سیو کریں
  await db.ref('chats/').push({
    refCode: refCode || 'guest',
    question: question,
    answer: finalAnswer,
    timestamp: Date.now()
  });

  res.json({
    success: true,
    appName: APP_NAME,
    isImage: false,
    answer: finalAnswer
  });
});

function isSimilar(str1, str2) {
  return str1.includes(str2) || str2.includes(str1);
}

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 ${APP_NAME} Production Server running on port ${PORT} with Full Knowledge Base!`);
});
