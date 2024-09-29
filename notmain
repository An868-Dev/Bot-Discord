import discord
from discord.ext import commands
import os
import asyncio
import yt_dlp
from dotenv import load_dotenv
import urllib.parse, urllib.request, re

def run_bot():
    load_dotenv()
    TOKEN = os.getenv('DISCORD_TOKEN')
    intents = discord.Intents.default()
    intents.message_content = True
    client = commands.Bot(command_prefix=".", intents=intents)

    queues = {}
    voice_clients = {}
    youtube_base_url = 'https://www.youtube.com/'
    youtube_results_url = youtube_base_url + 'results?'
    youtube_watch_url = youtube_base_url + 'watch?v='
    yt_dl_options = {"format": "bestaudio/best"}
    ytdl = yt_dlp.YoutubeDL(yt_dl_options)

    ffmpeg_options = {
        'before_options': '-reconnect 1 -reconnect_streamed 1 -reconnect_delay_max 5',
        'options': '-vn -filter:a "volume=0.25"'
    }

    @client.event
    async def on_ready():
        print(f'{client.user} is now jamming')

    async def play_next(ctx):
        if ctx.guild.id in queues and queues[ctx.guild.id]:
            link = queues[ctx.guild.id].pop(0)
            await play_song(ctx, link)
        elif ctx.guild.id in voice_clients:
            await voice_clients[ctx.guild.id].disconnect()
            del voice_clients[ctx.guild.id]


    async def play_song(ctx, link):
        try:
            if ctx.guild.id not in voice_clients:
                voice_client = await ctx.author.voice.channel.connect()
                voice_clients[ctx.guild.id] = voice_client

            loop = asyncio.get_event_loop()
            data = await loop.run_in_executor(None, lambda: ytdl.extract_info(link, download=False))

            song = data['url']
            player = discord.FFmpegOpusAudio(song, **ffmpeg_options)

            voice_clients[ctx.guild.id].play(player, after=lambda e: asyncio.run_coroutine_threadsafe(play_next(ctx), client.loop))
        except Exception as e:
            print(e)

    @client.command(name="play")
    async def play(ctx, *, link):
        if ctx.guild.id not in queues:
            queues[ctx.guild.id] = []
    
        queues[ctx.guild.id].append(link)
        await ctx.send("Sư đoàn 367 đã nhận được nhạc, đợi xog track sẽ phát liền!")

        if len(queues[ctx.guild.id]) == 1:
            await play_next(ctx)

    @client.command(name="clear_queue")
    async def clear_queue(ctx):
        if ctx.guild.id in queues:
            queues[ctx.guild.id].clear()
            await ctx.send("Queue cleared!")
        else:
            await ctx.send("There is no queue to clear")

    @client.command(name="pause")
    async def pause(ctx):
        try:
            voice_clients[ctx.guild.id].pause()
        except Exception as e:
            print(e)

    @client.command(name="resume")
    async def resume(ctx):
        try:
            voice_clients[ctx.guild.id].resume()
        except Exception as e:
            print(e)

    @client.command(name="stop")
    async def stop(ctx):
        try:
            voice_clients[ctx.guild.id].stop()
            await voice_clients[ctx.guild.id].disconnect()
            del voice_clients[ctx.guild.id]
        except Exception as e:
            print(e)

    @client.command(name="skip")
    async def skip(ctx):
        try:
            if ctx.guild.id in voice_clients:
                voice_clients[ctx.guild.id].stop()
                await ctx.send("Bài hát hiện tại đã bị bỏ qua. Đang phát bài tiếp theo...")
                await play_next(ctx)
            else:
                await ctx.send("Không có bài hát nào đang phát.")
        except Exception as e:
            print(e)

    @client.command(name='hello')
    async def hello(ctx):
        await ctx.send('Nghe nà baby!!')     

    client.run(TOKEN)
