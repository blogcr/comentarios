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

  // AÇÃO ADMIN
  if (data.acao === "ocultar") {
    sheet.getRange(data.linha, 7).setValue(false); // coluna ativo
    return jsonResponse({ success: true });
  }

  var dataHora = new Date();

  var postId = data.postId || "";
  var titulo = data.titulo || "";
  var autorPost = data.autorPost || "";
  var nome = (data.nome || "").trim();
  var comentario = (data.comentario || "").trim();
  var parentId = data.parentId || "";

  if (!nome || !comentario) {
    return jsonResponse({ error: "Preencha nome e comentário." });
  }

  sheet.appendRow([
    postId,      // A
    titulo,      // B
    autorPost,   // C
    nome,        // D
    comentario,  // E
    dataHora,    // F
    true,        // G
    parentId     // H
  ]);

  try {
    MailApp.sendEmail({
      to: EMAIL_DESTINO,
      subject: "Novo comentário no site",
      body:
        "Autor do post: " + autorPost + "\n" +
        "Título: " + titulo + "\n\n" +
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
      titulo: rows[i][1],
      autorPost: rows[i][2],
      nome: rows[i][3],
      comentario: rows[i][4],
      data: rows[i][5],
      ativo: rows[i][6],
      parentId: rows[i][7] || null
    });
  }

  if (e && e.parameter && e.parameter.modo === "admin") {

    resultado.sort(function(a, b) {
      return new Date(b.data) - new Date(a.data);
    });

    return jsonResponse(resultado.slice(0, 20));
  }

  return jsonResponse(
    resultado.filter(function(c) {
      return c.ativo === true;
    })
  );
}
