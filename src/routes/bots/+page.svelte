<script>
	import FriendTimeIcon from '$lib/images/bots/friend-time-icon.png';
	import QotdBotIcon from '$lib/images/bots/qotd-bot-icon.png';
	import BirthdayBotIcon from '$lib/images/bots/birthday-bot-icon.png';
</script>

<svelte:head>
	<title>Discord Bots | zones.arilyn.cc</title>
</svelte:head>

<h1>Discord Bots</h1>

<p class="text-muted mb-3">
	Arilyn Bots a small team of developers dedicated to creating fun and useful Discord bots. Here are
	some of our popular bots:
</p>

<h2><img src={FriendTimeIcon} alt="Friend Time Icon" class="bot-icon" /> Friend Time</h2>
<p>
	<a
		href="https://discord.com/oauth2/authorize?client_id=471091072546766849&scope=bot%20applications.commands&permissions=85056"
		target="_blank">Add Friend Time</a
	>
	|
	<a href="https://friend-time.arilyn.cc/" target="_blank" rel="noopener noreferrer">More Info</a>
</p>
<p>
	Friend Time helps your server coordinate times and events by converting times mentioned in chat
	between time zones! If you have a discord server with users across multiple countries and time
	zones then this is the bot for you! With Friend Time you can easily coordinate times between
	users.
</p>
<h2><img src={QotdBotIcon} alt="QOTD Bot Icon" class="bot-icon" /> QOTD Bot</h2>
<p>
	<a
		href="https://discord.com/oauth2/authorize?client_id=713586207119900693&scope=bot%20applications.commands&permissions=2252126231710720"
		target="_blank">Add QOTD Bot</a
	>
	|
	<a
		href="https://github.com/KevinNovak/QOTD-Bot-Docs/blob/master/README.md"
		target="_blank"
		rel="noopener noreferrer">More Info</a
	>
</p>
<p>
	QOTD Bot helps keep your community active by automatically posting questions (question of the
	day), facts, quotes, and more!
</p>

<h2><img src={BirthdayBotIcon} alt="Birthday Bot Icon" class="bot-icon" /> Birthday Bot</h2>
<p>
	<a
		href="https://discord.com/oauth2/authorize?client_id=656621136808902656&permissions=268921936&scope=bot%20applications.commands"
		target="_blank">Add Birthday Bot</a
	>
	|
	<a href="https://birthdaybot.scottbucher.dev/" target="_blank" rel="noopener noreferrer"
		>More Info</a
	>
</p>
<p>
	Celebrate birthdays, events, member anniversaries, server anniversaries, and more with the most
	customizable birthday bot!
</p>

<style>
	.bot-icon {
		width: 1.2em;
		height: 1.2em;
		vertical-align: middle;
		margin-right: 0.2em;
	}
</style>
