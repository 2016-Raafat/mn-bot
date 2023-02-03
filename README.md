# mn-bot

const { Client, GatewayIntentBits, Events } = require('discord.js');
const { joinVoiceChannel } = require('@discordjs/voice');
const db = require('pro.db')
const ms = require('ms');

const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMessages,
        GatewayIntentBits.MessageContent
    ]
});

require('./uptime');

const { prefix } = require('./config.json');

client.on('ready', () => {
    console.log(`Logged in as ${client.user.tag}!`);
}).login(process.env.token).catch(err => console.error(`${err}`))

client.on(Events.MessageCreate,async (message) => {
    if(!message.inGuild || message.author.bot) return false;
    if(message.content.startsWith(prefix + 'setchannel')) {
        if(message.author.id !== '487190339543629824') return false;
        let args = message.content.split(' ');

        let channel = message.mentions.channels.first() || message.guild.channels.cache.get(args[1])

        if(!channel) {
            return message.channel.send({content : `not Fuond channel`});
        } else {
            await db.set('T',{
                Guild : message.guildId,
                channel : message.channelId
            })
           return await message.channel.send({content : 'done'});
        }
    }
})

async function chack() {
    try {
    let data = await db.get('T');
    let channel = await client.channels.cache.get(data.channel);
    if(!channel) return false;
      
    const connection = joinVoiceChannel({
        channelId: channel.id,
        guildId: channel.guild.id,
        adapterCreator: channel.guild.voiceAdapterCreator,
    });
    } catch (er) {
        console.error(`${er}`);
    }
}

setInterval(async () => {
    chack()
}, ms('20s'))
