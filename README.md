<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8">
  <title>AI Ebook Generator</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 30px;
      background: #f9fafc;
      line-height: 1.6;
    }
    h1 { text-align: center; }
    form {
      background: #fff;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.1);
      margin-bottom: 20px;
    }
    label { font-weight: bold; display: block; margin-top: 10px; }
    input, textarea, select {
      width: 100%;
      padding: 8px;
      margin-top: 5px;
      border-radius: 5px;
      border: 1px solid #ddd;
    }
    button {
      margin-top: 15px;
      padding: 10px 15px;
      border: none;
      background: #4A90E2;
      color: white;
      font-size: 16px;
      border-radius: 6px;
      cursor: pointer;
    }
    button:hover { background: #357ABD; }
    #ebook {
      background: white;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.2);
      display: none;
    }
    .chapter {
      margin-top: 40px;
      page-break-before: always;
    }
    .chapter img {
      max-width: 100%;
      margin-top: 15px;
      border-radius: 6px;
    }
    #toc a {
      display: block;
      margin: 5px 0;
      color: #4A90E2;
      text-decoration: none;
    }
    #toc a:hover { text-decoration: underline; }
  </style>
</head>
<body>

<h1>📚 AI Ebook Generator</h1>

<form id="ebookForm">
  <label for="title">Tytuł ebooka:</label>
  <input type="text" id="title" required>

  <label for="topic">Tematyka:</label>
  <textarea id="topic" rows="2" required></textarea>

  <label for="chapters">Liczba rozdziałów (2–9):</label>
  <input type="number" id="chapters" min="2" max="9" required>

  <label for="pages">Liczba stron na rozdział:</label>
  <input type="number" id="pages" min="1" max="20" required>

  <label for="price">Cena ebooka (PLN):</label>
  <input type="number" id="price" min="0" required>

  <button type="submit">✨ Wygeneruj ebook</button>
</form>

<div id="ebook">
  <h2 id="ebookTitle"></h2>
  <p><em>Tematyka: <span id="ebookTopic"></span></em></p>
  <p><strong>Cena: <span id="ebookPrice"></span> PLN</strong></p>

  <h3>📑 Spis treści</h3>
  <div id="toc"></div>

  <div id="chaptersContent"></div>

  <button id="downloadPdf">📥 Pobierz jako PDF</button>
</div>

<!-- Biblioteka do PDF -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>

<script>
const form = document.getElementById('ebookForm');
const ebookDiv = document.getElementById('ebook');
const titleEl = document.getElementById('ebookTitle');
const topicEl = document.getElementById('ebookTopic');
const priceEl = document.getElementById('ebookPrice');
const tocEl = document.getElementById('toc');
const chaptersEl = document.getElementById('chaptersContent');
const downloadBtn = document.getElementById('downloadPdf');

form.addEventListener('submit', async function(e) {
  e.preventDefault();

  const reqData = {
    title: document.getElementById('title').value,
    topic: document.getElementById('topic').value,
    chapters: parseInt(document.getElementById('chapters').value),
    pages: parseInt(document.getElementById('pages').value),
    price: document.getElementById('price').value
  };

  // 🚀 Wyślij dane do backendu (musi działać backend Node.js z OpenAI)
  const response = await fetch("http://localhost:3000/generate-ebook", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(reqData)
  });

  const ebook = await response.json();

  // Render ebooka
  titleEl.textContent = ebook.title;
  topicEl.textContent = ebook.topic;
  priceEl.textContent = reqData.price;

  tocEl.innerHTML = '';
  chaptersEl.innerHTML = '';

  ebook.chapters.forEach(chap => {
    const chapId = 'chapter' + chap.number;
    const link = document.createElement('a');
    link.href = '#' + chapId;
    link.textContent = 'Rozdział ' + chap.number;
    tocEl.appendChild(link);

    const chapter = document.createElement('div');
    chapter.className = 'chapter';
    chapter.id = chapId;
    chapter.innerHTML = `
      <h3>Rozdział ${chap.number}</h3>
      <p>${chap.content.replace(/\n/g, '<br>')}</p>
      <img src="${chap.image}" alt="Ilustracja">
    `;
    chaptersEl.appendChild(chapter);
  });

  ebookDiv.style.display = 'block';
});

downloadBtn.addEventListener('click', function() {
  const opt = {
    margin:       0.5,
    filename:     'ebook.pdf',
    image:        { type: 'jpeg', quality: 0.98 },
    html2canvas:  { scale: 2 },
    jsPDF:        { unit: 'in', format: 'a4', orientation: 'portrait' }
  };
  html2pdf().set(opt).from(ebookDiv).save();
});
</script>

</body>
</html>
