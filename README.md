const TelegramBot = require('node-telegram-bot-api');
const { Configuration, OpenAIApi } = require("openai");
const fs = require('fs');
const ffmpeg = require('fluent-ffmpeg');
const { google } = require('googleapis');

// Replace with your actual OpenAI API key
const configuration = new Configuration({
  apiKey: 'YOUR_OPENAI_API_KEY', 
});
const openai = new OpenAIApi(configuration);

// Telegram Bot Token
const token = '7976609183:AAEaSMIbdzguwDNA5yCl0kY6I4ExODqsSM';

// Create a Telegram Bot instance
const bot = new TelegramBot(token, {polling: true});

// Welcome Message and Channel Link
const welcomeMessage = `Welcome! 👋 

Join our channel for updates and exclusive content: knownhub@ 

What would you like to do?`;

const channelJoinMessage = `Great! Now you can use me. 

Choose an option:`;

const buttons = [
  { text: 'Voice Generator' },
  { text: 'Video Generator' },
  { text: 'Photo Generator' },
  { text: 'Chat' },
];

const keyboard = { reply_markup: { keyboard: buttons, one_time_keyboard: true } };

// Handle Start Command
bot.onText(/\/start/, (msg) => {
  bot.sendMessage(msg.chat.id, welcomeMessage);
});

// Handle Channel Join
bot.on('message', (msg) => {
  if (msg.text === '/start') {
    return;
  }

  if (msg.text === 'I joined the channel') {
    bot.sendMessage(msg.chat.id, channelJoinMessage, keyboard);
  }
});

// Voice Generator
bot.onText(/Voice Generator/, async (msg) => {
  bot.sendMessage(msg.chat.id, 'Select gender:', {
    reply_markup: {
      keyboard: [['Male', 'Female']],
      one_time_keyboard: true,
    },
  });
});

bot.onText(/Male/, async (msg) => {
  bot.sendMessage(msg.chat.id, 'Enter the script:');
});

bot.on('message', async (msg) => {
  if (msg.text === 'Male' || msg.text === 'Female') {
    return;
  }

  try {
    const response = await openai.createAudio({
      input: msg.text,
      voice: 'alex', // Replace with desired male voice
    });

    const audioUrl = response.data.url;

    await bot.sendVoice(msg.chat.id, audioUrl);
    bot.sendMessage(msg.chat.id, 'Generate another voice? (Yes/No)');
  } catch (error) {
    console.error('Error generating male voice:', error);
    bot.sendMessage(msg.chat.id, 'Error generating voice. Please try again.');
  }
});

bot.onText(/Female/, async (msg) => {
  bot.sendMessage(msg.chat.id, 'Enter the script:');
});

bot.on('message', async (msg) => {
  if (msg.text === 'Male' || msg.text === 'Female') {
    return;
  }

  try {
    const response = await openai.createAudio({
      input: msg.text,
      voice: 'google_news_en', // Replace with desired female voice
    });

    const audioUrl = response.data.url;

    await bot.sendVoice(msg.chat.id, audioUrl);
    bot.sendMessage(msg.chat.id, 'Generate another voice? (Yes/No)');
  } catch (error) {
    console.error('Error generating female voice:', error);
    bot.sendMessage(msg.chat.id, 'Error generating voice. Please try again.');
  }
});

bot.onText(/Yes/, (msg) => {
  bot.sendMessage(msg.chat.id, 'Select gender:', {
    reply_markup: {
      keyboard: [['Male', 'Female']],
      one_time_keyboard: true,
    },
  });
});

bot.onText(/No/, (msg) => {
  bot.sendMessage(msg.chat.id, channelJoinMessage, keyboard);
});

// Video Generator
bot.onText(/Video Generator/, (msg) => {
  bot.sendMessage(msg.chat.id, 'Enter the video title:');
});

bot.on('message', async (msg) => {
  if (msg.text === 'Video Generator') {
    return;
  }

  try {
    // 1. Generate video script with OpenAI
    const response = await openai.createCompletion({
      model: 'text-davinci-003', // Choose a suitable model
      prompt: `Generate a short, engaging video script about "${msg.text}". Include a brief introduction, a main point, and a conclusion.`,
      max_tokens: 1024,
    });

    const script = response.data.choices[0].text;

    // 2. Generate voiceover with OpenAI
    const voiceoverResponse = await openai.createAudio({
      input: script,
      voice: 'alex', // Choose a voice
    });

    const voiceoverUrl = voiceoverResponse.data.url;

    // 3. Find and download stock footage/images
    // (Implement logic to search and download from stock footage/image websites)
    // For this example, we'll assume you have a function to get stock media URLs
