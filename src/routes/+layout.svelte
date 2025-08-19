<script lang="ts">
	import favicon from '$lib/assets/favicon.png';
	import navbarIcon from '$lib/assets/navbar-icon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<!-- Google tag (gtag.js) -->
	<script async src="https://www.googletagmanager.com/gtag/js?id=G-J5VV0NEQ6R"></script>
	<script>
		window.dataLayer = window.dataLayer || [];
		function gtag() {
			dataLayer.push(arguments);
		}
		gtag('js', new Date());

		gtag('config', 'G-J5VV0NEQ6R');
	</script>
	<title>zones.arilyn.cc</title>
	<link rel="icon" href={favicon} />
	<link
		href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css"
		rel="stylesheet"
		integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN"
		crossorigin="anonymous"
	/>
	<script
		src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"
		integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL"
		crossorigin="anonymous"
	></script>
	<link
		rel="stylesheet"
		href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css"
	/>
	<script
		async
		src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-2181679036667110"
		crossorigin="anonymous"
	></script>
</svelte:head>

<!-- Bootstrap Navbar -->
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
	<div class="container">
		<a class="navbar-brand" href="/">
			<img
				src={navbarIcon}
				alt="Navbar Icon"
				width="24"
				height="24"
				style="display: inline-block; vertical-align: text-bottom;"
			/>
			zones.arilyn.cc
		</a>
		<button
			class="navbar-toggler"
			type="button"
			data-bs-toggle="collapse"
			data-bs-target="#navbarNav"
			aria-controls="navbarNav"
			aria-expanded="false"
			aria-label="Toggle navigation"
		>
			<span class="navbar-toggler-icon"></span>
		</button>
		<div class="collapse navbar-collapse" id="navbarNav">
			<ul class="navbar-nav ms-auto">
				<li class="nav-item">
					<a class="nav-link" href="/">Map</a>
				</li>
				<li class="nav-item">
					<a class="nav-link" href="/list">List</a>
				</li>
				<li class="nav-item">
					<a class="nav-link" href="/convert">Convert</a>
				</li>
				<li class="nav-item">
					<a class="nav-link" href="/bots">Discord Bots</a>
				</li>
				<li class="nav-item">
					<a class="nav-link" href="/help">Help</a>
				</li>
				<li class="nav-item">
					<a class="nav-link" href="/changelog">Changelog</a>
				</li>
			</ul>
		</div>
	</div>
</nav>

<div class="container mt-4">
	{@render children?.()}
</div>

<!-- Footer -->
<footer class="bg-dark text-white py-5 mt-5">
	<div class="container">
		<div class="row">
			<div class="col-lg-4 mb-4">
				<h5 class="text-white">zones.arilyn.cc</h5>
				<p class="text-light">
					Time zone map and conversion tool designed to help users navigate global time differences.
				</p>
			</div>
			<div class="col-lg-3 col-md-6 mb-4">
				<h6 class="text-white">Quick Links</h6>
				<ul class="list-unstyled">
					<li><a href="/about" class="text-light text-decoration-none">About Us</a></li>
					<li><a href="/privacy" class="text-light text-decoration-none">Privacy Policy</a></li>
					<li><a href="/terms" class="text-light text-decoration-none">Terms of Service</a></li>
					<li><a href="/contact" class="text-light text-decoration-none">Contact Us</a></li>
					<li>
						<a
							href="https://www.paypal.com/donate/?cmd=_donations&business=EW389DYYSS4FC"
							target="_blank"
							class="text-light text-decoration-none">Donate</a
						>
					</li>
				</ul>
			</div>
			<div class="col-lg-4 col-md-12">
				<div class="text-md-end">
					<p class="mb-2 text-white">&copy; 2025 Arilyn Bots. All rights reserved.</p>
					<p class="text-light small">
						Built with SvelteKit |
						<a
							href="https://github.com/KevinNovak/zones-arilyn-cc"
							class="text-light"
							target="_blank"
							rel="noopener noreferrer">Source Code</a
						>
					</p>
					<p class="text-light small">IANA Time Zone Database 2025b</p>
				</div>
			</div>
		</div>
	</div>
</footer>
