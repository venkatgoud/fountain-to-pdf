<template>
  <iframe id="iframe_pdf" type="application/pdf" :src="pdf" width="100%" height="600"></iframe>
</template>

<script>
import * as jsPDF from "jspdf";
import "../BalooTammudu2-Regular-normal";
import "../Suranna-normal";
import { parse } from "fountain-parser";

/*eslint no-unused-vars: "off"*/

const defaultFont = "Courier";
// const teluguFont = "Suranna";
const teluguFont = "BalooTammudu2-Regular";
  
const isTelugu = str => /[\u{0c00}-\u{0C7F}]+/u.test(str);

const getFont = str => (isTelugu(str) ? teluguFont : defaultFont);

function createIndianPdf(str) {
  function setFont(str) {
    if (isTelugu(str)) {
      doc.setFont(getFont(str));
    }
  }

  function revertFont() {
    doc.setFont(defaultFont);
  }

  function drawCetnerLine() {
    doc
      .setDrawColor(128, 0, 0)
      .setLineWidth(1 / 144)
      .line(4.25, topMargin, 4.25, 11 - topMargin);
  }
  function writeText(str, width = maxPageWidth, lmargin = leftMargin) {
    const lines = doc.splitTextToSize(str, width);
    lines.forEach(line => {
      if (yPosition > maxPageHeight) {
        doc.addPage();
        drawCetnerLine();
        yPosition = topMargin;
      }
      setFont(line);
      doc.text(line, lmargin, yPosition);
      revertFont();
      yPosition += lineSpacing;
    });
  }

  function writeEmptyline() {
    if (yPosition > maxPageHeight) {
      doc.addPage();
      drawCetnerLine();
      yPosition = topMargin;
    }
    yPosition += lineSpacing;
  }

  function center(str, width = maxPageWidth) {
    const lines = doc.splitTextToSize(str, width);
    lines.forEach(line => {
      if (yPosition > maxPageHeight) {
        doc.addPage();
        // drawMargin(doc, leftMargin, topMargin);
        yPosition = topMargin;
      }
      doc.text(line, 4.25, yPosition, null, null, "center");
      yPosition += lineSpacing;
    });
  }

  function writeSceneHeading(str) {
    doc.setFontStyle("bold");
    writeText(str + "\n", 6, 1.5);
    doc.setFontStyle("normal");
  }

  function writeAction(str) {
    writeText(str + "\n", 2.5, 1.5);
  }

  function writeCharacter(str) {
    writeText(str, 3.3, 6);
  }

  function writeParenthetical(str) {
    writeText(str, 2, 4.8);
  }

  function writeDialog(str) {
    // console.log(str);
    writeText(str, 3.3, 4.8);
  }

  function writeTransition(str) {
    writeText(str + "\n", 1.5, 6);
  }

  function insertTitlePage() {
    doc.insertPage(1);
    center(scriptTitle);
    writeEmptyline();
    authorDetails.forEach(str => center(str));
    for (let index = 0; index < 6; index++) {
      writeEmptyline();
    }
    contactDetails.forEach(str => writeText(str));
  }

  function writeToken(token) {
    switch (token.type) {
      case "scene_heading":
        writeSceneHeading(token.text);
        break;
      case "action":
        writeAction(token.text);
        break;
      case "dialogue_begin":
        break;
      case "character":
        writeCharacter(token.text);
        break;
      case "parenthetical":
        writeParenthetical(token.text);
        break;
      case "dialogue":
        // doc.setFont("BalooTammudu2-Regular");
        writeDialog(token.text);
        // doc.setFont("Courier");
        break;
      case "dialogue_end":
        writeEmptyline();
        break;
      case "transition":
        writeTransition(token.text);
        break;
      case "centered":
        center(token.text);
        break;
      case "title":
        scriptTitle = token.text;
        break;
      case "credit":
      case "author":
      case "authors":
      case "source":
      case "notes":
        authorDetails.push(token.text);
        break;
      case "draft_date":
      case "date":
      case "contact":
      case "copyright":
        contactDetails.push(token.text);
        break;
      default:
        break;
    }
  }

  function fountainCallbacks(output) {
    const tokens = output.tokens;
    for (let i = 0; i < tokens.length; i++) {
      const token = tokens[i];
      writeToken(token);
      // console.log(token);
    }
    if (scriptTitle) {
      insertTitlePage();
    }
  }

  const options = {
    format: "letter",
    unit: "in"
  };

  var doc = new jsPDF(options);
  const fontSize = 12;
  const leftMargin = 1;
  const rightMargin = 1;
  const topMargin = 1;

  doc.setFont(defaultFont);
  doc.setFontSize(fontSize);

  const pageHeight = doc.internal.pageSize.height;
  const maxPageHeight = pageHeight - topMargin;
  const maxPageWidth = 8.5 - leftMargin - rightMargin;
  let yPosition = topMargin;

  // The height of a single line of text is simply the font size. A line of text at size 10 will take up 10 pt.
  const lineSpacing = (fontSize * 1.15) / 72;
  let authorDetails = [];
  let contactDetails = [];
  let scriptTitle;

  drawCetnerLine();
  parse(str, fountainCallbacks);

  return doc;
}

function createPdf(str, indian) {
  if (indian) {
    return createIndianPdf(str);
  }

  function setFont(str) {
    if (isTelugu(str)) {
      doc.setFont(getFont(str),"normal");
    }
  }

  function revertFont() {
    doc.setFont(defaultFont);
  }

  function drawMargin() {
    doc
      .setDrawColor(0, 1, 1)
      .setLineWidth(1 / 144)
      .line(leftMargin, topMargin, leftMargin, 11 - topMargin)
      .line(8.5 - leftMargin, topMargin, 8.5 - leftMargin, 11 - topMargin);
  }

  function writeText(str, width = maxPageWidth, lmargin = leftMargin) {
    const lines = doc.splitTextToSize(str, width);
    lines.forEach(line => {
      if (yPosition > maxPageHeight) {
        doc.addPage();
        // drawMargin(doc, leftMargin, topMargin);
        yPosition = topMargin;
      }
      setFont(line);
      doc.text(line, lmargin, yPosition);
      revertFont();
      yPosition += lineSpacing;
    });
  }

  function writeEmptyline() {
    if (yPosition > maxPageHeight) {
      doc.addPage();
      // drawMargin(doc, leftMargin, topMargin);
      yPosition = topMargin;
    }
    yPosition += lineSpacing;
  }

  function center(str, width = maxPageWidth) {
    const lines = doc.splitTextToSize(str, width);
    lines.forEach(line => {
      if (yPosition > maxPageHeight) {
        doc.addPage();
        // drawMargin(doc, leftMargin, topMargin);
        yPosition = topMargin;
      }
      setFont();
      doc.text(line, 4.25, yPosition, null, null, "center");
      revertFont();
      yPosition += lineSpacing;
    });
  }

  function writeSceneHeading(str) {
    doc.setFontStyle("bold");
    writeText(str + "\n", 6, 1.5);
    doc.setFontStyle("normal");
  }

  function writeAction(str) {
    writeText(str + "\n", 6, 1.5);
  }

  function writeCharacter(str) {
    writeText(str, 3.3, 4.2);
  }

  function writeParenthetical(str) {
    writeText(str, 2, 3.6);
  }

  function writeDialog(str) {     
    writeText(str, 3.3, 2.9);
  }

  function writeTransition(str) {
    writeText(str + "\n", 1.5, 6);
  }

  function insertTitlePage() {
    doc.insertPage(1);
    center(scriptTitle);
    writeEmptyline();
    authorDetails.forEach(str => center(str));
    for (let index = 0; index < 6; index++) {
      writeEmptyline();
    }
    contactDetails.forEach(str => writeText(str));
  }

  function writeToken(token) {
    switch (token.type) {
      case "scene_heading":
        writeSceneHeading(token.text);
        break;
      case "action":
        writeAction(token.text);
        break;
      case "dialogue_begin":
        break;
      case "character":
        writeCharacter(token.text);
        break;
      case "parenthetical":
        writeParenthetical(token.text);
        break;
      case "dialogue":         
        writeDialog(token.text);         
        break;
      case "dialogue_end":
        writeEmptyline();
        break;
      case "transition":
        writeTransition(token.text);
        break;
      case "centered":
        center(token.text);
        break;
      case "title":
        scriptTitle = token.text;
        break;
      case "credit":
      case "author":
      case "authors":
      case "source":
      case "notes":
        authorDetails.push(token.text);
        break;
      case "draft_date":
      case "date":
      case "contact":
      case "copyright":
        contactDetails.push(token.text);
        break;
      default:
        break;
    }
  }

  function fountainCallbacks(output) {
    const tokens = output.tokens;
    for (let i = 0; i < tokens.length; i++) {
      const token = tokens[i];
      writeToken(token);
    }
    if (scriptTitle) {
      insertTitlePage();
    }
  }

  // *italics*
  // **bold**
  // ***bold italics***
  // _underline_

  // One point is 1/72 of an inch. Thus, a 12-point font is 1/6 of an inch high

  // left -  8.5 × 11 inches
  // screenplay - letter paper, Courier 12pt
  // top and bottom margins - 1 inch
  // scene headings, action - left: 1.5 inch, right: 1 inch, width: 6
  // character - left: 4.2 inch, right: 1 inch, width: 3.3
  // Parenthetical - left: 3.6, right: 2.9, width: 2
  // dialog - left: 2.9 inches, right: 2.3, width: 3.3
  // transition - left: 6 inches, right: 1, width: 1.5
  const options = {
    format: "letter",
    unit: "in"
  };

  var doc = new jsPDF(options);
  const fontSize = 12;
  const leftMargin = 1;
  const rightMargin = 1;
  const topMargin = 1;

  doc.setFont(defaultFont);   
  doc.setFontSize(fontSize);

  const pageHeight = doc.internal.pageSize.height;
  const maxPageHeight = pageHeight - topMargin;
  const maxPageWidth = 8.5 - leftMargin - rightMargin;
  let yPosition = topMargin;
   
  // The height of a single line of text is simply the font size. A line of text at size 10 will take up 10 pt.
  const lineSpacing = (fontSize * 1.15) / 72;
  let authorDetails = [];
  let contactDetails = [];
  let scriptTitle;

  // drawMargin();

  parse(str, fountainCallbacks);

  return doc;
}

export default {
  name: "PDFScreenplay",
  props: {
    fountain: String,
    indian: Boolean
  },
  computed: {
    pdf: function() {
      return createPdf(this.fountain, this.indian).output("datauristring");
    }
  }
};
</script>
