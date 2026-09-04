
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel Mestre - Centros, Médias & Timer</title>
    <style>
        body {
            background-color: #0b0e11;
            color: #eaecef;
            font-family: monospace;
            padding: 10px;
            margin: 0;
        }
        h2 { text-align: center; color: #f0b90b; font-size: 16px; margin-bottom: 5px; }
        .timer-topo {
            text-align: center;
            font-size: 14px;
            color: #0ecb81;
            margin-bottom: 10px;
            font-weight: bold;
        }
        .alerta-box {
            background-color: #1e2329;
            border: 2px solid #f0b90b;
            padding: 10px;
            text-align: center;
            font-size: 14px;
            font-weight: bold;
            margin-bottom: 10px;
            border-radius: 5px;
        }
        .linha-tempo {
            background-color: #181a20;
            border-bottom: 1px solid #2b313a;
            padding: 8px;
            margin-bottom: 6px;
            border-radius: 4px;
            font-size: 12px;
        }
        .topo-linha {
            display: flex;
            justify-content: space-between;
            margin-bottom: 4px;
            font-weight: bold;
        }
        .medias-linha {
            color: #848e9c;
            font-size: 11px;
            word-break: break-all;
        }
        .comp { color: #0ecb81; }
    </style>
</head>
<body>

    <h2>BTUSD - CENTROS & MÉDIAS MESTRES</h2>
    <div id="relogio-m1" class="timer-topo">Vela M1 Fechando em: --:--</div>
    
    <div id="status-sinal" class="alerta-box" style="color: #f0b90b;">
        🔍 CONECTANDO NA BINANCE...
    </div>

    <div id="painel-tempos">
        <!-- Dados carregam aqui -->
    </div>

<script>
const tempos = [
    { nome: "M1", intervalo: "1m" },
    { nome: "M5", intervalo: "5m" },
    { nome: "M15", intervalo: "15m" },
    { nome: "M30", intervalo: "30m" },
    { nome: "H1", intervalo: "1h" },
    { nome: "H4", intervalo: "4h" }
];

const periodosMa = [19, 38, 97, 191, 383, 575, 979];

function calcularCentro(velas) {
    let maxima = -Infinity;
    let minima = Infinity;
    velas.forEach(v => {
        let alta = parseFloat(v[2]);
        let baixa = parseFloat(v[3]);
        if (alta > maxima) maxima = alta;
        if (baixa < minima) minima = baixa;
    });
    return (maxima + minima) / 2;
}

function calcularMedia(velas, periodo) {
    if (velas.length < periodo) return null;
    let soma = 0;
    for (let i = velas.length - periodo; i < velas.length; i++) {
        soma += parseFloat(velas[i][4]);
    }
    return soma / periodo;
}

async function atualizarPainel() {
    let htmlGeral = "";
    let sinalGlobal = "⚖️ AGUARDANDO CRUZAMENTO";
    let corSinal = "#f0b90b";

    for (let t of tempos) {
        try {
            let res = await fetch(`https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=${t.interval}&limit=1000`);
            let dados = await res.json();

            let centro = calcularCentro(dados);
            let precoAtual = parseFloat(dados[dados.length - 1][4]);

            let mediasTexto = "";
            periodosMa.forEach(p => {
                let maVal = calcularMedia(dados, p);
                if (maVal) {
                    mediasTexto += `MA${p}: ${maVal.toFixed(1)} | `;
                }
            });

            if (precoAtual > centro) {
                sinalGlobal = `🚀 [${t.nome}] PREÇO ACIMA DO CENTRO -> COMPRA FORTE!`;
                corSinal = "#0ecb81";
            } else {
                sinalGlobal = `📉 [${t.nome}] PREÇO ABAIXO DO CENTRO -> VENDA FORTE!`;
                corSinal = "#f6465d";
            }

            htmlGeral += `
                <div class="linha-tempo">
                    <div class="topo-linha">
                        <span>${t.nome} | Atual: $ ${precoAtual.toFixed(2)}</span>
                        <span>Comp: <span class="comp">$ ${centro.toFixed(2)}</span></span>
                    </div>
                    <div class="medias-linha">${mediasTexto || "Calculando médias..."}</div>
                </div>
            `;
        } catch (e) {
            htmlGeral += `<div class="linha-tempo"><b>${t.nome}</b>: Erro ao carregar</div>`;
        }
    }

    document.getElementById("painel-tempos").innerHTML = htmlGeral;
    
    let caixaSinal = document.getElementById("status-sinal");
    caixaSinal.innerText = sinalGlobal;
    caixaSinal.style.color = corSinal;
    caixaSinal.style.borderColor = corSinal;
}

// Timer regressivo do M1 rodando a cada segundo
function atualizarTimer() {
    const agora = new Date();
    let segundosRestantes = 59 - agora.getSeconds();
    let segundosFormatados = segundosRestantes < 10 ? "0" + segundosRestantes : segundosRestantes;
    document.getElementById("relogio-m1").innerText = `Vela M1 Fechando em: 00:${segundosFormatados}`;
}

setInterval(atualizarPainel, 5000);
setInterval(atualizarTimer, 1000);

atualizarPainel();
atualizarTimer();
</script>

</body>
</html>
