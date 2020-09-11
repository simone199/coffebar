# coffebar
coffebot

import discord
import random
from discord.ext import commands, tasks
from discord.ext.commands import Bot
from discord import utils
from itertools import cycle
import asyncio
import os

client = commands.Bot(command_prefix = 'c?')
client.remove_command('help')
status = cycle(['c?help ' , 'coffebar'])

@client.event                         
async def on_ready():
    change_status.start()
    print(f"Con l'id: [ID utente client]")
    print ("Start in corso")

@tasks.loop(seconds=10)
async def change_status():
    await client.change_presence(activity=discord.Activity(type=discord.ActivityType.watching, name="to know the list of commands type c?help"))



@client.command()
async def ping(ctx):
    await ctx.send(f'Pong 🏓  {round(client.latency * 1000)}ms')


@client.command(aliases=['8ball', 'test'])
async def _8ball(ctx, *, question):
    embed = discord.Embed(color=0x1560bd)
    responses = ['🟢    Probabilmente ',
                '🔴 Probabilmente no ',]
    await ctx.send(f'❓ Domanda: {question}\nRisposta: {random.choice(responses)}')

@client.command()
async def avatar(ctx, member: discord.Member = None):
    if not member:
        member = ctx.message.author
    embed = discord.Embed(color=0x1560bd)
    embed.title = "👥Avatar Utente"
    embed.set_image(url=member.avatar_url)
    await ctx.send(embed=embed)

@client.command(pass_context=True)
@commands.has_permissions(administrator=True)
async def clear(ctx, arg: int):
    if not arg:
        embed = discord.Embed(
            color=discord.Colour.blue()
        )
        embed.set_author(
            name="[🛡️] Security Guard Specifica quante messaggi vuoi cancellare",
            icon_url="https://cdn.discordapp.com/attachments/640563710104043530/730639329453670420/DuscePeppe_FRIULI.png"
        )
        await ctx.send(embed=embed)
        return
    embed = discord.Embed(
        color=discord.Colour.blue()
    )
    embed.set_author(
        name=f'[🛡️] Security Guard Ho cancellato ufficialmente:  {arg} messaggi',
        icon_url=f'{ctx.author.avatar_url}'
    )
    await ctx.channel.purge(limit=arg+1)
    await ctx.send(embed=embed)
    embed = discord.Embed(
        color=discord.Colour.blue()
    )
    embed.set_author(
        name=f'{ctx.author._user}[🛡️] Security Guard ha cancellato: {arg}',
        icon_url=f'{ctx.author.avatar_url}'
    )
    embed.add_field(
        name='[🛡️] Security Guard Messaggi cancellati da:',
        value=f'{ctx.author._user}',
        inline=True
    )
    embed.add_field(
        name='Quantità:',
        value=f'{arg}',
        inline=True
    )
#    channel = client.get_channel(729553772547932190)
    await discord.channel.send(embed=embed)
@clear.error
async def clear_error(ctx, error):
    if isinstance(error, commands.CheckFailure):
        embed = discord.Embed(
            color=discord.Colour.blue()
        )
        embed.set_author(
            name="**[🛡️] Security Guard Non ti è permesso cancellare i messaggi**",
        )
        await ctx.send(embed=embed)

client.command()


@client.command()
@commands.has_permissions(administrator=True)
async def ban(ctx, member: discord.Member=None,*,arg=' [🛡️] Security Guard Motivo non specificato'):
    embed = discord.Embed(
        colour = discord.Colour.red()
    )
    await ctx.message.delete()
    await ctx.member.ban()
    await ctx.send (f' [🛡️] Security Guard è stato bannato per: {arg}')
    await ctx.send(embed=embed)




@ban.error
async def ban_error(ctx, error):
    if isinstance(error, commands.CheckFailure):
        embed = discord.Embed(
            color=discord.Colour.red()
        )
        embed.set_author(
            name="Non ti è permesso bannare!",
        )
        await ctx.send(embed=embed)



@client.command(aliases=['ub'])
@commands.has_permissions(ban_members=True)
async def unban(ctx,*,member):
        banned_users = await ctx.guild.bans()
        member_name, member_disc = member.split('#')

        for banned_entry in banned_users:
            user = banned_entry.user

        if(user.name, user.discriminator)==(member_name,member_disc):

                await ctx.guild.unban(user)
                await ctx.send(member_name +"**[🛡️] Security Guard Utente è stato sbannato**")
                return

        await ctx.send(member+" war not found")

@client.command()
@commands.has_permissions(administrator=True)
async def kick(ctx, member : discord.Member, *, reason=None):
    await member.kick(reason=reason)
    await ctx.send(f'Kicked {member.mention}')

@kick.error
async def kick_error(ctx, error):
    if isinstance(error, commands.CheckFailure):
        embed = discord.Embed(
            color=discord.Colour.blue()
        )
        embed.set_author(
            name="**[🛡️] Security Guard non ti è permesso kickare**",
        )
        await ctx.send(embed=embed)



@client.command()
async def userinfo(ctx, member: discord.Member):

    roles = [role for role in member.roles]

    embed = discord.Embed(colour=member.color, timestamp=ctx.message.created_at)

    embed.set_author(name=f"info untente - {member}")
    embed.set_thumbnail(url=member.avatar_url)
    embed.set_footer(text=f"Commando eseguito da {ctx.author}", icon_url=ctx.author.avatar_url)

    embed.add_field(name="**ID:**", value=member.id)
    embed.add_field(name="**nome utente:**", value=member.display_name)

    embed.add_field(name="**account creato:**", value=member.created_at.strftime("%a, %#d %B %Y, %I:%M %p UTC"))
    embed.add_field(name="**entrato nel server:**", value=member.joined_at.strftime("%a, %#d %B %Y, %I:%M %p UTC"))

    embed.add_field(name=f"**Ruoli:** ({len(roles)})", value=" ".join([role.mention for role in roles]))
    embed.add_field(name="**ruoli alti:**", value=member.top_role.mention)

    embed.add_field(name=" **è un Bot ?:**", value=member.bot)

    await ctx.send(embed=embed)

@client.command()
async def serverinfo(ctx, guild: discord.Guild = None):
    guild = ctx.guild if not guild else guild
    embed = discord.Embed(title=f"Server info {guild}", description="coffeebar", timestamp = ctx.message.created_at, color=discord.Color.red())
    embed.set_thumbnail(url=guild.icon_url)
    embed.add_field(name="**Fondatore**", value=guild.owner, inline=False)
    embed.add_field(name="**Server creato**:", value=guild.created_at, inline=False)
    embed.add_field(name="**Descrizione**:", value= guild.description, inline=False)
    embed.add_field(name="**numero membri**:", value= guild.member_count, inline=False)
    embed.add_field(name="**numero Canali**:", value= len(guild.channels), inline=False)
    embed.add_field(name="**numero ruoli**:", value=len(guild.roles), inline=False)
    embed.add_field(name="**Numero Boost**:", value= guild.premium_subscription_count, inline=False)
    embed.add_field(name="**Emoji:**", value=guild.emoji_limit, inline=False)
    embed.set_footer(text=f"Commando eseguito da {ctx.author}", icon_url = ctx.author.avatar_url)
    await ctx.send(embed=embed)

@client.command(pass_context=True)
async def help(ctx):
    author = ctx.message.author

    embed = discord.Embed(
        colour = discord.Colour.blue()
    )


    embed.set_author(name='comandi del bot')
    embed.add_field(name='c?clear', value='`Elimina un  messaggio specifico`', inline=False)
    embed.add_field(name='c?ban', value='`Banna un utente specifico`', inline=False)
    embed.add_field(name='c?avatar', value='`Ottieni un utente avatar specifico`', inline=False)
    embed.add_field(name='c?8ball', value='`messaggio casuale`', inline=False)
    embed.add_field(name='c?ping', value='`mostra il ping del bot `', inline=False)
    embed.add_field(name='c?kick', value='`Kicka un utente specifico`', inline=False)
    embed.add_field(name='c?unban', value='`sbanna un utente specifico`', inline=False)
    embed.add_field(name='c?mute', value='`Muta un utente specifico`', inline=False)
    embed.add_field(name='c?unmute', value='`smutta un utente specifico `', inline=False)
    embed.add_field(name='c?userinfo', value='`Informazioni su specifico  utente`', inline=False)
    embed.add_field(name='c?serverinfo', value='`Informazioni sul server `', inline=False)
    embed.add_field(name='c?invite', value='`manda il link per invitare il bot `', inline=False)
    embed.add_field(name='c?say', value='`ti permette di replicare um messagio `', inline=False)
    
    

    await author.send(embed=embed)

@client.command()
async def invite(ctx):
    await ctx.send(f'https://discord.com/oauth2/authorize?client_id=732887104647987210&scope=bot&permissions=1812462847bot')

@client.command()
async def say(ctx, *, msg):
    await ctx.message.delete()
    await ctx.send("{}" .format(msg))












client.run('TOKEN')

