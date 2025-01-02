import telebot
import json

# Replace with your bot's API token
bot = telebot.TeleBot("7692421097:AAFxwcKCPfe5DI7PLAPZcFOSlys2_l2dj7s")

# Store user IDs in a list (to send the broadcast message to all users)
user_ids = []

# Load user IDs from a file (if saved before)
def load_user_ids():
    try:
        with open('user_ids.json', 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        return []

# Save user IDs to a file
def save_user_ids():
    with open('user_ids.json', 'w') as f:
        json.dump(user_ids, f)

# Command to start the bot and add the user ID to the list
@bot.message_handler(commands=["start"])
def start(message):
    user_id = message.from_user.id
    if user_id not in user_ids:
        user_ids.append(user_id)
        save_user_ids()
    bot.reply_to(message, "Welcome! You can use /broadcast to send a broadcast message to all users.")

# Command to send a broadcast message with an image
@bot.message_handler(commands=["broadcast"])
def send_broadcast(message):
    if message.from_user.id != 7163960299:  # Replace with your Telegram user ID to restrict access
        bot.reply_to(message, "You are not authorized to use this command.")
        return

    # Extract the broadcast message from the user's input (excluding "/broadcast")
    broadcast_message = message.text[len('/broadcast '):]

    # Check if the message contains an image (from a reply)
    if message.reply_to_message and message.reply_to_message.photo:
        photo = message.reply_to_message.photo[-1].file_id  # Get the highest quality image
    else:
        photo = None

    if broadcast_message or photo:
        # Send the broadcast message to all users
        for user_id in user_ids:
            try:
                if photo:
                    bot.send_photo(user_id, photo, caption=broadcast_message)
                else:
                    bot.send_message(user_id, broadcast_message)
            except Exception as e:
                print(f"Error sending message to {user_id}: {e}")

        # Send the broadcast message to the group
        group_chat_id = '-1002400719738'  # Replace with your group chat ID
        try:
            if photo:
                bot.send_photo(group_chat_id, photo, caption=broadcast_message)
            else:
                bot.send_message(group_chat_id, broadcast_message)
            bot.reply_to(message, "Broadcast message sent to all users and the group!")
        except Exception as e:
            print(f"Error sending message to group: {e}")
    else:
        bot.reply_to(message, "Please provide a message and/or image to broadcast.")

# Start the bot
bot.polling()
