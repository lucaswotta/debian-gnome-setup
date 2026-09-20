# LibreOffice no estilo do Microsoft Office

**Status:** concluída · **Escopo:** depende do perfil de uso. Serve a quem troca arquivos com o Microsoft Office

[← Fase 5](05-aplicativos.md) · [Roteiro](../02-roteiro.md) · [Fase 6 →](06-gnome.md)

Objetivo: fazer o LibreOffice se comportar como o Microsoft Office para quem recebe e envia arquivos dele: faixa de opções em abas, ícones no estilo do Office, gravação em `.docx`, `.xlsx` e `.pptx`, idioma pt-BR, folha branca, fontes compatíveis e modelos com os padrões do Microsoft 365 (fonte, margens e espaçamento).

O guia tem duas partes: a aparência e o formato de gravação, e os padrões do Microsoft 365.

## Aparência e formato de gravação

Objetivo: faixa de opções em abas, ícones no estilo do Office e gravação em `.docx`, `.xlsx` e `.pptx`, mantendo o layout dos documentos.
Os pacotes de idioma, ajuda, dicionário e as fontes compatíveis já vêm do [lote 1](05-aplicativos.md#lote-1-base-de-desenvolvimento-fontes-e-docker).

Pela interface:

- *Exibir > Interface do usuário...*, escolher *Em abas* e clicar em *Aplicar a todos*.
- *Ferramentas > Opções > LibreOffice > Exibir > Estilo dos ícones*: Colibre (SVG).
- *Ferramentas > Opções > Carregar/Salvar > Geral*: em "Sempre salvar como", o formato do Office para cada tipo de documento, e
  desmarcar o aviso ao gravar fora do formato ODF.

Pelo perfil, com o LibreOffice **fechado** (ele reescreve o arquivo ao sair):

```bash
pgrep -cx soffice.bin                                # 0: LibreOffice fechado
soffice --headless --norestore --terminate_after_init   # cria o perfil, se ainda não existir
cp -a ~/.config/libreoffice/4/user/registrymodifications.xcu{,.bak}
```

Itens acrescentados ao `registrymodifications.xcu` (uma linha `<item>` para cada um, antes do `</oor:items>`):

| Ajuste | Caminho e propriedade | Valor |
|---|---|---|
| Faixa em abas | `/org.openoffice.Office.UI.ToolbarMode/Applications/Writer`, `Calc`, `Impress` e `Draw`, propriedade `Active` | `notebookbar.ui` |
| Layout da faixa | `/org.openoffice.Office.UI.ToolbarMode`, propriedades `ActiveWriter`, `ActiveCalc`, `ActiveImpress` e `ActiveDraw` | `notebookbar.ui` |
| Ícones | `/org.openoffice.Office.Common/Misc`, propriedade `SymbolStyle` | `colibre_svg` |
| Formato do Word | `/org.openoffice.Setup/Office/Factories/com.sun.star.text.TextDocument`, propriedade `ooSetupFactoryDefaultFilter` | `Office Open XML Text` |
| Formato do Excel | `.../com.sun.star.sheet.SpreadsheetDocument`, mesma propriedade | `Calc MS Excel 2007 XML` |
| Formato do PowerPoint | `.../com.sun.star.presentation.PresentationDocument`, mesma propriedade | `Impress MS PowerPoint 2007 XML` |
| Sem aviso de formato | `/org.openoffice.Office.Common/Save/Document`, propriedade `WarnAlienFormat` | `false` |
| Sem linhas de margem na folha | `/org.openoffice.Office.Writer/Content/Display`, propriedade `TextBoundaries` | `false` |
| Régua vertical | `/org.openoffice.Office.Writer/Layout/Window`, propriedade `VerticalRuler` | `true` |

Os três formatos padrão de gravação só persistem quando gravados pela API de configuração (`ConfigurationUpdateAccess`, como no exemplo do
modelo padrão mais abaixo). O mesmo valor escrito à mão no `registrymodifications.xcu` pode ser descartado e o *Salvar como* volta a sugerir ODF.

Cada item segue este formato:

```xml
<item oor:path="/org.openoffice.Office.UI.ToolbarMode/Applications/Writer"><prop oor:name="Active" oor:op="fuse"><value>notebookbar.ui</value></prop></item>
```

Como conferir:

```bash
# Fidelidade de fontes: gera um documento com Calibri, Cambria e Arial e confere o resultado
soffice --headless --convert-to docx teste.fodt && soffice --headless --convert-to pdf teste.fodt
unzip -p teste.docx word/document.xml | grep -o 'w:ascii="[^"]*"' | sort -u   # nomes da Microsoft
pdffonts teste.pdf                                                             # Carlito, Caladea e Arial
```

Ao abrir o Writer, a faixa deve mostrar abas (Arquivo, Página Inicial, Inserir, Layout...), e o *Salvar como* deve sugerir `.docx`.

Como desfazer: com o LibreOffice fechado, `cp -a ~/.config/libreoffice/4/user/registrymodifications.xcu.bak ~/.config/libreoffice/4/user/registrymodifications.xcu`.

Observações:

- **Nomes de fonte no arquivo:** o `.docx` grava `Calibri` e `Cambria`, e o LibreOffice as exibe com o Carlito e o Caladea, de mesmas métricas.
  O Word de quem receber usa as originais, e o layout se mantém.
- **Valor da chave `Active`:** é o valor do modo (`notebookbar.ui`), e não o nome exibido no menu (`Tabbed`). Um texto que não seja o
  valor de um modo é aceito pela API, mas ignorado, e o LibreOffice abre no modo clássico.
- **Valores dos modos:** `Default` (menus clássicos), `Single`, `Sidebar`, `notebookbar.ui` (em abas), `notebookbar_compact.ui` (em abas,
  mais baixa), `notebookbar_groupedbar_compact.ui`, `notebookbar_groupedbar_full.ui`, `notebookbar_single.ui` e `notebookbar_groups.ui`.
- **Descobrir a chave certa:** faça a escolha pela interface e compare o `registrymodifications.xcu` antes e depois. O LibreOffice grava
  exatamente o que ele lê. Ler a chave de volta pela API (UNO) confirma que o valor existe, mas não que o programa o reconhece.
- **Ícones:** o Colibre é o tema do LibreOffice no Windows e o mais próximo do Office. A variante escura é a `colibre_dark_svg`.
- **Fonte e estilos de documentos novos:** ver [Padrões do Microsoft 365](#padrões-do-microsoft-365).
- **Processo aberto:** `pgrep -f soffice.bin` casa com a própria linha de comando do `pgrep`. Use `pgrep -x soffice.bin`.
- **Documentos complexos:** se a fidelidade em documentos muito formatados não bastar, o ONLYOFFICE (Flatpak) reproduz melhor o Office, com a
  interface em faixa de opções.

## Padrões do Microsoft 365

Objetivo: aproximar a aparência do que se produz no LibreOffice do que o Microsoft 365 novo produz, mantendo o layout dos documentos.

**Formato padrão do Word.** O filtro `Word 2007–365` (`MS Word 2007 XML`) grava `compatibilityMode=12`, e o Word abre o arquivo em modo de
compatibilidade. O filtro `Word 2010–365` (`Office Open XML Text`) grava `15` e abre normalmente. Por isso o padrão é o segundo:

```bash
soffice --headless --convert-to 'docx:MS Word 2007 XML'   --outdir a arquivo.fodt
soffice --headless --convert-to 'docx:Office Open XML Text' --outdir b arquivo.fodt
unzip -p a/arquivo.docx word/settings.xml | grep -o 'compatibilityMode"[^>]*'   # w:val="12"
unzip -p b/arquivo.docx word/settings.xml | grep -o 'compatibilityMode"[^>]*'   # w:val="15"
```

**Fundo do documento em modo escuro.** Com o tema escuro, a folha também fica escura. Para manter a folha branca:
*Ferramentas > Opções > LibreOffice > Cores do aplicativo > Fundo do documento*, em branco. A escolha vale para todos os aplicativos e é gravada em
`/org.openoffice.Office.UI/ColorScheme/ColorSchemes/...['COLOR_SCHEME_LIBREOFFICE_AUTOMATIC']/DocColor`, na propriedade `Dark` (`16777215`).

**Fonte Aptos.** A Aptos, padrão do Microsoft 365 novo, é proprietária e não existe para Linux. Sem ela, o sistema a troca por Noto Sans, bem mais
larga, e os documentos recebidos mudam de paginação. A regra abaixo escolhe a fonte instalada de largura mais próxima.
Largura do mesmo parágrafo, com a Calibri (Carlito) em 100%, lida dos arquivos de fonte:

| Fonte | Largura |
|---|---|
| Liberation Sans Narrow | 90,3% |
| Carlito (Calibri) | 100% |
| Liberation Sans (Arial) | 110,1% |
| Noto Sans | 115,5% |
| DejaVu Sans | 124,7% |

```bash
mkdir -p ~/.config/fontconfig/conf.d
cat > ~/.config/fontconfig/conf.d/60-aptos-substituta.conf <<'EOF'
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
  <alias binding="same"><family>Aptos</family><prefer><family>Liberation Sans</family></prefer></alias>
  <alias binding="same"><family>Aptos Display</family><prefer><family>Liberation Sans</family></prefer></alias>
  <alias binding="same"><family>Aptos Narrow</family><prefer><family>Liberation Sans Narrow</family></prefer></alias>
  <alias binding="same"><family>Aptos Mono</family><prefer><family>Liberation Mono</family></prefer></alias>
</fontconfig>
EOF
fc-cache -f && fc-match Aptos          # Liberation Sans
```

O arquivo `.docx` continua gravando o nome `Aptos`, então o Word de quem receber usa a fonte original. A substituta só vale na tela e no PDF gerado aqui.
É uma aproximação: sem a Aptos instalada, não dá para medir a diferença exata.

**Idioma.** Interface, formatos e ortografia em português do Brasil. Pacotes: `libreoffice-l10n-pt-br`, `libreoffice-help-pt-br`,
`hunspell-pt-br`, `hyphen-pt-br` e `mythes-pt-br`. Chaves gravadas pela API de configuração:

| Ajuste | Caminho e propriedade | Valor |
|---|---|---|
| Idioma da interface | `/org.openoffice.Setup/L10N`, propriedade `ooLocale` | `pt-BR` |
| Formatos de data, número e moeda | `/org.openoffice.Setup/L10N`, propriedade `ooSetupSystemLocale` | `pt-BR` |
| Idioma padrão dos documentos (ortografia) | `/org.openoffice.Office.Linguistic/General`, propriedade `DefaultLocale` | `pt-BR` |

Os modelos abaixo também gravam `pt-BR` no estilo padrão de cada aplicativo. Scripts UNO devem rodar com o idioma do usuário (`LANG=pt_BR.UTF-8`).
Com `LC_ALL=C`, o LibreOffice trata a sessão como inglesa e os nomes de planilha, de estilo e da interface saem em inglês.

**Grade do Calc e paleta de cores.** A grade fica em cinza claro (`#D4D4D4`), como no Excel, nas propriedades `Light` e `Dark` do nó `CalcGrid`
do esquema de cores. A paleta *Office* (arquivo `~/.config/libreoffice/4/user/config/Office.soc`) traz as cores do tema do Office 2023 em diante, com as cinco
variações de cada uma (mais claro 80%, 60% e 40%, mais escuro 25% e 50%), calculadas em HSL, e as dez cores padrão. Está selecionada em
`/org.openoffice.Office.Common/UserColors`, propriedade `PaletteName` (`Office`):

| Cor do tema | Valor |
|---|---|
| Texto 2 | `#0E2841` |
| Destaque 1 a 6 | `#156082`, `#E97132`, `#196B24`, `#0F9ED5`, `#A02B93`, `#4EA72E` |
| Hiperlink e hiperlink visitado | `#467886` e `#96607D` |

**Modelos padrão.** Os modelos abaixo aproximam os padrões do Word, do Excel e do PowerPoint do Microsoft 365 novo. São criados por um script UNO num
perfil temporário e definidos como padrão pela API de configuração:

| Estilo | Valor |
|---|---|
| Padrão (Word) | Aptos 11 pt, 8 pt depois do parágrafo, entrelinha 1,15 |
| Título 1, 2 e 3 | Aptos Display 20 pt, Aptos Display 16 pt e Aptos 14 pt, cor `#0F4761`, sem negrito |
| Título e Subtítulo | Aptos Display 28 pt, e Aptos 14 pt na cor `#595959` |
| Página | A4, margens de 2,54 cm nos quatro lados (predefinição *Normal* do Word) |

Modelo do Excel (`Excel365.ots`):

| Item | Valor |
|---|---|
| Célula padrão | Aptos Narrow 11 pt, `pt-BR` |
| Planilha | uma, chamada `Planilha1` |
| Coluna | 1,693 cm (8,43 caracteres, 64 px) |
| Página | A4, margens de 1,91 cm em cima e embaixo e de 1,78 cm nas laterais (predefinição *Normal* do Excel), sem cabeçalho e sem rodapé |

Modelo do PowerPoint (`Impress365.otp`):

| Item | Valor |
|---|---|
| Slide | 33,867 x 19,05 cm (16:9, o tamanho padrão do PowerPoint) |
| Título | Aptos Display 44 pt, preto, centralizado na vertical, em uma caixa de 29,21 x 3,68 cm a 2,33 cm da borda esquerda |
| Texto | Aptos 28, 24, 20 e 18 pt (níveis 1 a 5 e seguintes), entrelinha 90%, 10 pt antes do parágrafo |
| Marcadores | `•` em Arial, tamanho 100%, recuo de 0,635 cm e mais 1,27 cm por nível |
| Data, rodapé e número | Aptos 12 pt, cinza `#898989` |
| Slide 1 | *Slide de título*: título de 60 pt centralizado e ancorado embaixo, subtítulo de 24 pt centralizado |
| Formas e caixas de texto | Aptos 18 pt |

Os valores de posição do PowerPoint vêm do tema padrão do Office (`12192000 x 6858000` EMU, com 1 EMU = 1/360 de centésimo de milímetro).

```python
# soffice --headless --accept="socket,host=127.0.0.1,port=2005;urp;" &   (com um perfil temporário, se preferir)
import uno, os
from com.sun.star.beans import PropertyValue
def pv(n, v): p = PropertyValue(); p.Name = n; p.Value = v; return p
ctx = uno.getComponentContext().ServiceManager.createInstanceWithContext("com.sun.star.bridge.UnoUrlResolver", uno.getComponentContext()) \
        .resolve("uno:socket,host=127.0.0.1,port=2005;urp;StarOffice.ComponentContext")
desktop = ctx.ServiceManager.createInstanceWithContext("com.sun.star.frame.Desktop", ctx)
def espacamento(pct):
    ls = uno.createUnoStruct("com.sun.star.style.LineSpacing"); ls.Mode = 0; ls.Height = pct; return ls
doc = desktop.loadComponentFromURL("private:factory/swriter", "_blank", 0, (pv("Hidden", True),))
estilos = doc.StyleFamilies.getByName("ParagraphStyles")
estilos.getByName("Standard").setPropertyValue("CharFontName", "Aptos")
estilos.getByName("Standard").setPropertyValue("CharHeight", 11.0)
estilos.getByName("Standard").setPropertyValue("ParaBottomMargin", 282)          # 8 pt, em centésimos de mm
estilos.getByName("Standard").setPropertyValue("ParaLineSpacing", espacamento(115))
# ... Título 1 a 3, Título, Subtítulo e página, como na tabela acima ...
doc.storeToURL("file://" + os.path.expanduser("~/.config/libreoffice/4/user/template/Word365.ott"), (pv("FilterName", "writer8_template"),))
doc.close(True)
```

Definir os modelos como padrão, com o LibreOffice fechado ou por uma sessão UNO:

```python
cp = ctx.ServiceManager.createInstanceWithContext("com.sun.star.configuration.ConfigurationProvider", ctx)
modelos = {"com.sun.star.text.TextDocument": "Word365.ott",
           "com.sun.star.sheet.SpreadsheetDocument": "Excel365.ots",
           "com.sun.star.presentation.PresentationDocument": "Impress365.otp"}
for fabrica, arquivo in modelos.items():
    no = cp.createInstanceWithArguments("com.sun.star.configuration.ConfigurationUpdateAccess",
            (pv("nodepath", "/org.openoffice.Setup/Office/Factories/" + fabrica),))
    no.setPropertyValue("ooSetupFactoryTemplateFile", "file://" + os.path.expanduser("~/.config/libreoffice/4/user/template/" + arquivo))
    no.commitChanges()
```

Como conferir, abrindo um documento novo de cada tipo:

```python
d = desktop.loadComponentFromURL("private:factory/swriter", "_blank", 0, (pv("Hidden", True),))
e = d.StyleFamilies.getByName("ParagraphStyles").getByName("Standard")
print(e.CharFontName, e.CharHeight)      # Aptos 11.0
# Calc: "scalc", estilo de célula "Default", estilo de página "Default"
# Impress: "simpress", d.StyleFamilies.getByName(d.MasterPages.getByIndex(0).Name).getByName("title")
```

Como desfazer: apague a chave `ooSetupFactoryTemplateFile` de cada fábrica pela mesma API (valor vazio), remova os arquivos de
`~/.config/libreoffice/4/user/template/` e o `Office.soc`, e restaure o perfil com o backup `registrymodifications.xcu.bak`.

Observações:

- **Editar o perfil à mão nem sempre funciona:** o LibreOffice descartou o valor de `ooSetupFactoryTemplateFile` escrito direto no
  `registrymodifications.xcu`. Pela API de atualização de configuração, o mesmo valor persistiu. Confirme sempre abrindo um documento novo.
- **Validar pela API:** ler uma chave de volta não prova que o programa a usa. Abra um documento novo e leia os estilos.
- **Estilos derivados:** os estilos que não foram ajustados (lista, legenda, índice) continuam em Liberation. Só os estilos da tabela acima seguem o Word.
- **Margens e valores dos estilos:** as margens seguem a predefinição *Normal* documentada pela Microsoft (2,54 cm no Word). Um documento em branco criado
  no Word do trabalho é a referência exata: a tag `w:pgMar` do `document.xml` traz as margens, e dá para importar os estilos dele em vez de recriá-los.
  O mesmo vale para `<sheetFormatPr>` e `<pageMargins>` do `.xlsx` e para `sldSz` do `.pptx`.
- **Régua vertical:** o menu *Exibir > Régua* liga só a horizontal. A vertical fica em *Ferramentas > Opções > LibreOffice Writer > Exibir > Régua vertical*
  e vem desligada, então as margens de cima e de baixo só aparecem ao ligá-la.
- **Linhas de margem:** os cantos que o Writer desenha na folha são os *limites do texto*. Ficam desligados pela chave `TextBoundaries`
  (*Exibir > Limites do texto* faz o mesmo pela interface).
- **Altura das linhas do Calc:** o LibreOffice a calcula pela fonte e resulta em 0,487 cm, contra 15 pt (0,529 cm) do Excel. Não foi fixada porque
  uma altura fixa desliga o ajuste automático das linhas com quebra de texto.
- **Impress, família de estilos:** os estilos de apresentação (`title`, `outline1` a `outline9`, `subtitle`) ficam numa família com o nome do slide mestre
  (`Padrão` em português).
- **Impress, marcadores:** `replaceByIndex` em `NumberingRules` só aceita a sequência com `uno.invoke(regras, "replaceByIndex", (i, uno.Any("[]com.sun.star.beans.PropertyValue", valores)))`
  e os campos curtos (`NumberingType`, `Adjust`, `StartWith`, `BulletRelSize`, `SymbolTextDistance`) como `uno.Any("short", valor)`.
- **Layouts do PowerPoint:** o Impress tem um só título e um só corpo por slide mestre. O *Slide de título* fica aplicado só ao primeiro slide do modelo.
