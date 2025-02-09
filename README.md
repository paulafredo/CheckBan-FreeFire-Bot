# Bot Discord Free Fire Info

## Description
Ce bot Discord permet d'obtenir des informations détaillées sur les joueurs de Free Fire, de vérifier leur statut de bannissement et de récupérer des données sur les guildes et chefs de guildes. Il utilise l'API RapidAPI pour récupérer les informations et l'API officielle de Free Fire pour vérifier les bannissements.

## Fonctionnalités
- 🔍 **Obtenir des informations sur un joueur** via la commande `/get_info`
- 🚨 **Vérifier si un joueur est banni** via la commande `/check_ban`
- 📜 **Afficher les détails de la guilde** et de son leader
- 🌍 **Support multi-régions**

## Installation
### Prérequis
- Python 3.8+
- Un bot Discord enregistré ([Créer un bot ici](https://discord.com/developers/applications))
- Un compte RapidAPI avec une clé d'API valide

### Étapes d'installation
1. **Cloner le projet**
   ```bash
   git clone https://github.com/ton-repo/discord-bot-freefire.git
   cd discord-bot-freefire
   ```
2. **Installer les dépendances**
   ```bash
   pip install -r requirements.txt
   ```
3. **Configurer les variables d'environnement**
   Créer un fichier `.env` et ajouter :
   ```env
   APPLICATION_ID=ton_application_id
   TOKEN=ton_token_discord
   RAPIDAPI_KEY=ta_cle_rapidapi
   RAPIDAPI_HOST=ton_hote_rapidapi
   ```
4. **Lancer le bot**
   ```bash
   python bot.py
   ```

## Commandes disponibles
| Commande       | Description |
|---------------|-------------|
| `/get_info` `<UID>` | Obtenir les informations d'un joueur Free Fire |
| `/check_ban` `<UID>` | Vérifier si un joueur est banni |

## Code Source
```python
import discord
import aiohttp
import os
from discord.ext import commands
from discord import app_commands
from dotenv import load_dotenv

load_dotenv()

intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix="!", intents=intents)

APPLICATION_ID = os.getenv("APPLICATION_ID")
TOKEN = os.getenv("TOKEN")
RAPIDAPI_KEY = os.getenv("RAPIDAPI_KEY")
RAPIDAPI_HOST = os.getenv("RAPIDAPI_HOST")

HEADERS = {
    'x-rapidapi-key': RAPIDAPI_KEY,
    'x-rapidapi-host': RAPIDAPI_HOST,
}

@bot.event
async def on_ready():
    print(f"Le bot est connecté en tant que {bot.user}")
    await bot.tree.sync()

@bot.tree.command(name="get_info", description="Obtenez des informations sur un joueur de Free Fire.")
@app_commands.describe(uid="UID à vérifier")
async def get_info_command(interaction: discord.Interaction, uid: str):
    await interaction.response.defer()
    data_info = await get_player_info(uid)

    if 'error' in data_info:
        await interaction.followup.send(f"❌ {data_info['error']}")
        return

    embed = discord.Embed(
        title="📜 Informations du joueur",
        description=f"""
🔹 **Pseudo:** {data_info['nickname']}
🔹 **UID:** {data_info['accountId']}
🔹 **Niveau:** {data_info['level']}
🔹 **Région:** {data_info['region']}
🔹 **Likes:** {data_info['liked']}
🔹 **Dernière connexion:** <t:{data_info['lastLoginAt']}:R>
🔹 **Signature:** {data_info['socialInfo']}
        """,
        color=0x0099ff,
        timestamp=discord.utils.utcnow(),
    )

    if data_info['avatar_image_url']:
        embed.set_image(url=data_info['avatar_image_url'])

    await interaction.followup.send(embed=embed)

async def get_player_info(player_id):
    try:
        if not player_id.isdigit():
            return {'error': 'Player ID doit être un entier valide.'}

        url = f"https://id-game-checker.p.rapidapi.com/ff-player-info/{player_id}/SG"

        async with aiohttp.ClientSession() as session:
            async with session.get(url, headers=HEADERS) as response:
                if response.status != 200:
                    return {'error': f"Erreur API: {response.status}"}

                response_data = await response.json()
                status = response_data.get('status')

                if status == 200:
                    data = response_data.get('data', {})
                    return {
                        'accountId': data.get('basicInfo', {}).get('accountId', 'N/A'),
                        'nickname': data.get('basicInfo', {}).get('nickname', 'N/A'),
                        'level': data.get('basicInfo', {}).get('level', 'N/A'),
                        'region': data.get('basicInfo', {}).get('region', 'N/A'),
                        'liked': data.get('basicInfo', {}).get('liked', 'N/A'),
                        'lastLoginAt': data.get('basicInfo', {}).get('lastLoginAt', 'N/A'),
                        'socialInfo': data.get('socialInfo', {}).get('signature', 'N/A'),
                        'avatar_image_url': data.get('profileInfo', {}).get('clothes', {}).get('images', [None])[0],
                    }
                else:
                    return {'error': f"Erreur API: {status}"}

    except Exception as e:
        return {'error': f"Une erreur s'est produite : {str(e)}"}

bot.run(TOKEN)
```

## Licence
Ce projet est sous licence MIT.


