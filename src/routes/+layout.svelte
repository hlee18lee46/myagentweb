<script>
	import Header from '$lib/header/Header.svelte';
  import { webVitals } from '$lib/vitals';
  import { browser } from '$app/env';
  import { page } from '$app/stores';
  import '../app.css';

  let analyticsId = import.meta.env.VERCEL_ANALYTICS_ID;

  $: if (browser && analyticsId) {
    webVitals({
      path: $page.url.pathname,
      params: $page.params,
      analyticsId
    })
  }
</script>

<Header />

<main>
	<slot />
</main>

<footer>
    <div class="social-icons">
        <a href="https://github.com/hlee18lee46" target="_blank">
            <img src="/icons/github.png" alt="GitHub" />
        </a>
        <a href="https://devpost.com/hlee18lee46" target="_blank">
            <img src="/icons/devpost.png" alt="Devpost" />
        </a>
        <a href="https://www.linkedin.com/in/hanknowstech" target="_blank">
            <img src="/icons/linkedin.png" alt="LinkedIn" />
        </a>
        <a href="/resume.pdf" target="_blank">
            <img src="/icons/resume.png" alt="Resume" />
        </a>
    </div>
</footer>


<style>
	main {
		flex: 1;
		display: flex;
		flex-direction: column;
		padding: 1rem;
		width: 100%;
		max-width: 1024px;
		margin: 0 auto;
		box-sizing: border-box;
	}

	footer {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		padding: 40px;
	}

	footer a {
		font-weight: bold;
	}

	@media (min-width: 480px) {
		footer {
			padding: 40px 0;
		}
	}

	    footer {
        margin-top: 20px;
        text-align: center;
    }

    .social-icons {
        display: flex;
        justify-content: center;
        gap: 15px;
        margin-top: 10px;
    }

    .social-icons img {
        width: 30px;
        height: 30px;
        transition: transform 0.2s;
    }

    .social-icons img:hover {
        transform: scale(1.2);
    }
</style>
