var SHEET_ID = "1JGcM6A2JxUCAHbr_d8XOFbgftpj0idUkEffXUHNrud4";
var EMAIL_DESTINO = "contato@carlosromero.com.br";


function getSheet() {
  return SpreadsheetApp.openById(SHEET_ID).getSheets()[0];
}

function jsonResponse(obj) {
  return ContentService
    .createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}


/* =========================
   NOVO COMENTÁRIO
========================= */
function doPost(e) {

  var sheet = getSheet();

  if (!e || !e.postData) {
    return jsonResponse({ error: "Requisição inválida." });
  }

  var data = JSON.parse(e.postData.contents);

  // 👉 AÇÃO ADMIN (OCULTAR)
  if (data.acao === "ocultar") {

    var id = data.linha;
    sheet.getRange(id, 5).setValue(false); // coluna "ativo"

    return jsonResponse({ success: true });
  }

  var dataHora = new Date();
  var nome = (data.nome || "").trim();
  var comentario = (data.comentario || "").trim();
  var parentId = data.parentId || "";
  var postId = data.postId || "";

  if (!nome || !comentario) {
    return jsonResponse({ error: "Preencha nome e comentário." });
  }

  sheet.appendRow([
    postId,
    nome,
    comentario,
    dataHora,
    true,
    parentId
  ]);

  try {
    MailApp.sendEmail({
      to: EMAIL_DESTINO,
      subject: parentId ? "Nova resposta no site" : "Novo comentário no site",
      body:
        "Post: " + postId + "\n\n" +
        "Nome: " + nome + "\n\n" +
        "Comentário: " + comentario
    });
  } catch (err) {}

  return jsonResponse({ success: true });
}


/* =========================
   LISTAGEM
========================= */
function doGet(e) {

  var sheet = getSheet();
  var rows = sheet.getDataRange().getValues();
  var resultado = [];

  for (var i = 1; i < rows.length; i++) {

    resultado.push({
      linha: i + 1,
      postId: rows[i][0],
      nome: rows[i][1],
      comentario: rows[i][2],
      data: rows[i][3],
      ativo: rows[i][4],
      parentId: rows[i][5] || null
    });
  }

  // MODO ADMIN
  if (e && e.parameter && e.parameter.modo === "admin") {

    resultado.sort(function(a, b) {
      return new Date(b.data) - new Date(a.data);
    });

    return jsonResponse(resultado.slice(0, 20));
  }

  // SITE PÚBLICO (somente ativos)
  var publicos = resultado.filter(function(c) {
    return c.ativo === true;
  });

  return jsonResponse(publicos);
}
