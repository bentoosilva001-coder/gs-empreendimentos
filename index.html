// ============================================================
// LUMI BOT — Grupo LumiNex
// ============================================================
// Três peças trabalhando juntas:
// 1. Baileys: conecta esse código ao SEU WhatsApp (via QR Code),
//    exatamente como o WhatsApp Web faz.
// 2. Groq (Whisper): transcreve áudios recebidos em texto — de graça,
//    dentro do limite gratuito.
// 3. Anthropic SDK: manda o texto (digitado OU transcrito) pro Claude,
//    junto com a personalidade do Lumi, e recebe a resposta.
// ============================================================

import 'dotenv/config';
import makeWASocket, { useMultiFileAuthState, DisconnectReason, downloadMediaMessage } from '@whiskeysockets/baileys';
import qrcodeTerminal from 'qrcode-terminal';
import QRCode from 'qrcode';
import Anthropic from '@anthropic-ai/sdk';
import pino from 'pino';
import { SYSTEM_PROMPT } from './lumi-persona.js';

// Cliente da Anthropic — usa a chave que você vai colocar no arquivo .env
const anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// Memória de conversa, EM RAM (na memória do processo, não salva em
// disco). Isso significa: se o bot reiniciar, esquece as conversas
// em andamento. Pra um primeiro teste está ótimo; se quiser memória
// permanente depois, dá pra trocar por um banco de dados (etapa futura).
const conversationHistory = new Map();

// Guarda só as últimas N mensagens por conversa, pra não gastar
// tokens (e dinheiro) demais mandando o histórico inteiro toda hora.
const MAX_HISTORY = 20;

// ============================================================
// FASE 5 — PROTEÇÃO DE CUSTO E USO
// ============================================================

// NÍVEL 8: preço REAL do Claude Haiku 4.5 (setembro/2026):
// $1 por milhão de tokens de ENTRADA, $5 por milhão de SAÍDA.
// Dividimos por 1 milhão pra ter o preço "por token", que é a
// unidade que o response.usage devolve depois de cada chamada.
const PRICE_PER_INPUT_TOKEN = 1 / 1_000_000;
const PRICE_PER_OUTPUT_TOKEN = 5 / 1_000_000;

// Configuráveis pelo .env, com um valor padrão caso não seja definido.
const DAILY_BUDGET_USD = parseFloat(process.env.DAILY_BUDGET_USD || '1.00');
const MAX_MESSAGES_PER_HOUR = parseInt(process.env.MAX_MESSAGES_PER_HOUR || '15', 10);

// Estado do "cofre": quanto já foi gasto hoje, e quando foi o
// último reset (pra zerar automaticamente à meia-noite).
let dailySpend = 0;
let lastResetDate = new Date().toDateString();

// Histórico de horários de mensagem por pessoa, pra calcular o
// rate limit — Map de "número" → lista de timestamps (em ms).
const messageTimestamps = new Map();

// Reseta o gasto do dia se a data mudou desde a última mensagem.
function resetDailySpendIfNewDay() {
  const today = new Date().toDateString();
  if (today !== lastResetDate) {
    console.log(`🔄 Novo dia — resetando gasto (ontem: $${dailySpend.toFixed(4)})`);
    dailySpend = 0;
    lastResetDate = today;
  }
}

// Verifica se essa pessoa já mandou mensagem demais na última hora.
// Retorna true se PODE continuar, false se estourou o limite.
function checkRateLimit(from) {
  const now = Date.now();
  const oneHourAgo = now - 60 * 60 * 1000;

  const timestamps = messageTimestamps.get(from) || [];
  // Mantém só os horários da última hora (descarta os mais antigos)
  const recentTimestamps = timestamps.filter((t) => t > oneHourAgo);

  recentTimestamps.push(now);
  messageTimestamps.set(from, recentTimestamps);

  return recentTimestamps.length <= MAX_MESSAGES_PER_HOUR;
}

// ============================================================
// FASE 4 — SUPORTE A MENSAGENS DE ÁUDIO
// ============================================================

// NÍVEL 8: Claude não "escuta" áudio diretamente pela API — só lê
// texto, imagem e PDF. Por isso usamos um serviço à parte (Groq,
// rodando o Whisper) só pra transformar áudio em texto. Depois que
// vira texto, o resto do fluxo é EXATAMENTE igual a uma mensagem
// digitada — o Claude nem sabe que aquilo começou como áudio.
async function transcribeAudio(buffer) {
  // FormData e Blob já vêm prontos no Node 18+, sem precisar instalar nada.
  const formData = new FormData();
  formData.append('file', new Blob([buffer]), 'audio.ogg');
  formData.append('model', 'whisper-large-v3-turbo'); // versão rápida, dentro do plano gratuito
  formData.append('language', 'pt'); // já avisamos que é português, acelera e melhora a precisão

  const res = await fetch('https://api.groq.com/openai/v1/audio/transcriptions', {
    method: 'POST',
    headers: { Authorization: `Bearer ${process.env.GROQ_API_KEY}` },
    body: formData,
  });

  if (!res.ok) {
    const errText = await res.text();
    throw new Error(`Groq respondeu ${res.status}: ${errText}`);
  }

  const data = await res.json();
  return data.text.trim();
}

async function startBot() {
  // useMultiFileAuthState salva a "sessão" numa pasta local (auth_info/).
  // Depois do primeiro login (escanear o QR), o bot reconecta sozinho
  // sem pedir escaneamento de novo — até você deletar essa pasta ou
  // desconectar manualmente pelo celular.
  const { state, saveCreds } = await useMultiFileAuthState('auth_info');

  const sock = makeWASocket({
    auth: state,
    // Desliga os logs internos bem verbosos do Baileys; deixamos só
    // os nossos console.log próprios, mais fáceis de acompanhar.
    logger: pino({ level: 'silent' }),
  });

  // --- Evento de conexão: aqui aparece o QR Code e o status ---
  sock.ev.on('connection.update', (update) => {
    const { connection, lastDisconnect, qr } = update;

    if (qr) {
      console.log('\n📱 Escaneie esse QR Code no WhatsApp > Aparelhos conectados:\n');
      qrcodeTerminal.generate(qr, { small: true });

      // NÍVEL 7: além do QR no terminal (que às vezes fica pequeno
      // ou cortado dependendo do tamanho da janela), também salvamos
      // como um arquivo de imagem de verdade — muito mais fácil de
      // abrir em tela cheia e escanear sem erro.
      QRCode.toFile('qr.png', qr, { width: 500 }, (err) => {
        if (!err) console.log('🖼️  Também salvei como imagem: abra o arquivo qr.png (apareceu na barra lateral do VS Code) e escaneie ele em tela cheia.\n');
      });
    }

    if (connection === 'close') {
      // Verifica o motivo da desconexão. Se NÃO foi um logout manual
      // (você desconectando pelo celular), o bot tenta reconectar sozinho.
      const statusCode = lastDisconnect?.error?.output?.statusCode;
      const shouldReconnect = statusCode !== DisconnectReason.loggedOut;
      console.log('❌ Conexão fechada.', shouldReconnect ? 'Reconectando...' : 'Você fez logout — rode o script de novo pra reconectar.');
      if (shouldReconnect) startBot();
    } else if (connection === 'open') {
      console.log('✅ Lumi conectado ao WhatsApp! Aguardando mensagens...\n');
    }
  });

  // Sempre que as credenciais mudam (ex: primeira conexão), salva
  // no disco pra não precisar escanear o QR de novo depois.
  sock.ev.on('creds.update', saveCreds);

  // --- Evento de mensagem recebida: aqui mora o "cérebro" ---
  sock.ev.on('messages.upsert', async ({ messages }) => {
    const msg = messages[0];

    // Ignora mensagens vazias e mensagens que O PRÓPRIO BOT enviou
    // (senão ele ficaria "respondendo a si mesmo" infinitamente).
    if (!msg.message || msg.key.fromMe) return;

    const from = msg.key.remoteJid;
    let text = msg.message.conversation || msg.message.extendedTextMessage?.text || '';
    const isAudio = !!msg.message.audioMessage;

    if (isAudio) {
      if (!process.env.GROQ_API_KEY) {
        console.log(`🎤 Áudio recebido de ${from}, mas GROQ_API_KEY não está configurada — ignorando.`);
        await sock.sendMessage(from, { text: 'Ainda não consigo ouvir áudios 🙏 Pode mandar em texto?' });
        return;
      }
      console.log(`🎤 Áudio recebido de ${from}, transcrevendo...`);
      try {
        // downloadMediaMessage baixa o arquivo de áudio criptografado
        // do WhatsApp e devolve já descriptografado, pronto pra usar.
        const buffer = await downloadMediaMessage(msg, 'buffer', {});
        text = await transcribeAudio(buffer);
        console.log(`📝 Transcrição: "${text}"`);
      } catch (err) {
        console.error('⚠️ Erro ao transcrever áudio:', err.message);
        await sock.sendMessage(from, { text: 'Não consegui entender esse áudio 😕 Pode tentar de novo, ou em texto?' });
        return;
      }
    }

    if (!text) return; // ignora figurinha, imagem sem legenda etc (por enquanto)

    console.log(`📩 Mensagem de ${from}${isAudio ? ' (via áudio)' : ''}: ${text}`);

    resetDailySpendIfNewDay();

    // --- Proteção 1: limite de mensagens por pessoa ---
    if (!checkRateLimit(from)) {
      console.log(`🚫 ${from} atingiu o limite de ${MAX_MESSAGES_PER_HOUR} mensagens/hora — ignorando (sem chamar a IA, sem custo).`);
      await sock.sendMessage(from, {
        text: 'Você mandou bastante mensagem em pouco tempo! Me dá uns minutinhos e volta a conversar 🙂',
      });
      return; // sai ANTES de gastar um único token
    }

    // --- Proteção 2: teto de gasto diário ---
    if (dailySpend >= DAILY_BUDGET_USD) {
      console.log(`🚫 Orçamento diário estourado ($${dailySpend.toFixed(4)} / $${DAILY_BUDGET_USD}) — resposta padrão, sem chamar a IA.`);
      await sock.sendMessage(from, {
        text: 'Nosso atendimento automático está com alta demanda no momento! Em breve alguém da equipe te responde por aqui 🙏',
      });
      return;
    }

    // Recupera (ou cria) o histórico dessa conversa específica
    const history = conversationHistory.get(from) || [];
    history.push({ role: 'user', content: text });

    try {
      // Mostra "digitando..." no WhatsApp do cliente, pra parecer
      // mais natural (não é instantâneo feito robô).
      await sock.sendPresenceUpdate('composing', from);

      const response = await anthropic.messages.create({
        model: 'claude-haiku-4-5-20251001', // modelo rápido e barato, ideal pra chat
        max_tokens: 400,
        system: SYSTEM_PROMPT,
        messages: history,
      });

      const reply = response.content[0].text;

      // Calcula o custo REAL dessa chamada específica, usando os
      // números que a própria API devolve (quantos tokens ela
      // realmente processou), não uma estimativa.
      const { input_tokens, output_tokens } = response.usage;
      const callCost = (input_tokens * PRICE_PER_INPUT_TOKEN) + (output_tokens * PRICE_PER_OUTPUT_TOKEN);
      dailySpend += callCost;

      console.log(`💰 Custo dessa mensagem: $${callCost.toFixed(5)} | Total hoje: $${dailySpend.toFixed(4)} / $${DAILY_BUDGET_USD}`);

      // Guarda a resposta no histórico e mantém só as últimas mensagens
      history.push({ role: 'assistant', content: reply });
      conversationHistory.set(from, history.slice(-MAX_HISTORY));

      await sock.sendMessage(from, { text: reply });
      console.log(`🤖 Lumi respondeu: ${reply}\n`);
    } catch (err) {
      console.error('⚠️ Erro ao chamar a IA:', err.message);
      await sock.sendMessage(from, {
        text: 'Desculpa, tive um probleminha técnico agora 🙏 Pode repetir sua mensagem?',
      });
    }
  });
}

startBot();
