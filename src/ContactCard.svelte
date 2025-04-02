<!-- contactcard.svelte -->
<style>
	article {
		display: flex;
		flex-direction: column;
		padding: 1em;
		border-radius: 5px;
		border: 5px solid #2f363d; /* Thicker, darker border */
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); /* Maintain shadow at bottom */
	}

	h1 {
		color: #2f363d;
		margin: 0;
		line-height: 0.9em;
		transition: color 0.2s ease;
	}

	h1:hover {
		color: #170370;
	}

	h2 {
		font-size: 1.2em;
		margin: 0;
		transition: color 0.2s ease;
	}

	h2:hover {
		color: #170370;
	}

	header {
		margin-bottom: 10px;
	}
	
	.detail-container {
		display: flex;
		flex-direction: column;
		padding: 5px 0px;
		border-left: 7px solid rgb(22, 3, 109);
		overflow: hidden;
		margin-bottom: 8px;
		transition: all 0.2s ease;
	}

	.detail-container:hover {
		background-color: rgba(23, 3, 112, 0.05);
		border-left-width: 10px;
	}

	.detail-divider {
		border-top: 1px solid #e0e0e0;
		margin: 5px 10px 8px 10px;
		transition: border-color 0.2s ease;
	}

	.detail-container:hover + .detail-divider,
	.detail-container:hover ~ .detail-divider {
		border-top-color: rgba(23, 3, 112, 0.2);
	}

	.bottom-top-divider {
		border-top: 5px solid #170370;
		margin: 5px 10px 8px 10px;
	}

	.detail-content {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.detail-label {
		font-size: 0.8em;
		color: #4d4b4b;
		margin-left: 10px;
		margin-bottom: 2px;
		text-transform: uppercase;
		font-weight: bold;
		transition: color 0.2s ease;
	}

	.detail-container:hover .detail-label {
		color: #170370;
	}

	.detail-container p {
		margin: 5px 10px;
		font-size: 1em;
		word-break: break-word;
		flex-grow: 1;
		transition: color 0.2s ease;
	}

	.detail-container:hover p {
		color: #170370;
	}

	.detail-container img {
		width: 24px;
		height: 24px;
		margin-right: 10px;
	}

	a img {
		transition: all 0.1s ease-in-out;
	}
	a:hover img{
		transform: scale(1.25);
	}

	div.button-container {
		display: flex;
		justify-content: flex-end;
	}

	button {
		margin: 0 5px;
		padding: 0.2em 1.5em;
		font-weight: 600;

		border: 0;
		border-radius: 5px;

		cursor: pointer;

		transition: all 0.1s ease-in;
		background-color: whitesmoke;
	}

	button.download {
		background-color: #D64550;
	}

	button.edit {
		margin-right: auto;
		padding: 0.2em 0.5em;
	}

	/*Contact Info*/
	.contact-meta {
		display:flex;
		flex-direction: column;
		justify-content: space-evenly;
		width: 100%;
	}
</style>
<script>
	import Toast from 'svelte-toast'
	import QRCode from './QRCode.svelte'
	import { navigate } from "svelte-routing";

	export let canEdit = false;
	export let qCard;
	$: selfLink = `https://qcard.link${qCard.toViewUrl()}`

	function downloadVCard(){
		var fileName = qCard.name + ".vcf";
		var downloadElement = document.createElement("a");
		downloadElement.setAttribute("href", "data:text/vcard:charset=utf-8," + encodeURIComponent(qCard.toVCardString()));
		downloadElement.setAttribute("download", fileName);
		downloadElement.style.display = "none";

		document.body.appendChild(downloadElement);
		downloadElement.click();
		document.body.removeChild(downloadElement);
	}

	function shareQCard(){
		if (navigator.share) {
			navigator.share({
				title: `${qCard.name}'s QCard`,
				text: `Here's ${qCard.name}'s contact information`,
				url: selfLink,
			})
				.then(() => console.log('Successful share'))
				.catch((error) => console.log('Error sharing', error));
		}
	}

	function copySelfLink(){
		var el = document.createElement('textarea');
		el.value = selfLink;
		el.setAttribute('readonly', '');
		el.style = { display: 'none', position: 'absolute'};
		document.body.appendChild(el);
		el.select();
		document.execCommand('copy');
		document.body.removeChild(el);

		var toast = new Toast();
		toast.success(qCard.name + '\'s ' + 'QCard Copied');
	}

	function editQCard(){
		navigate(`/create/#${qCard.toEncodedString()}`)
	}

	function handleWhatsAppClick(phoneNumber, event) {
		event.preventDefault();
		const isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
		
		if (isMobile) {
			// Store visibility state before attempting to open WhatsApp
			let visibilityChange;
			if (typeof document.hidden !== "undefined") {
				visibilityChange = "visibilitychange";
			} else if (typeof document.msHidden !== "undefined") {
				visibilityChange = "msvisibilitychange";
			} else if (typeof document.webkitHidden !== "undefined") {
				visibilityChange = "webkitvisibilitychange";
			}
			
			// Set a flag to track if WhatsApp opened successfully
			let whatsAppOpened = false;
			
			// Listen for visibility changes (happens when app opens)
			const visibilityHandler = () => {
				if (document.hidden || document.msHidden || document.webkitHidden) {
					whatsAppOpened = true;
				}
			};
			
			if (visibilityChange) {
				document.addEventListener(visibilityChange, visibilityHandler, false);
			}
			
			// Try to open WhatsApp
			window.location.href = `whatsapp://send?phone=${phoneNumber}`;
			
			// Check after a delay if WhatsApp didn't open
			setTimeout(() => {
				if (visibilityChange) {
					document.removeEventListener(visibilityChange, visibilityHandler, false);
				}
				
				// if (!whatsAppOpened) {
				// 	alert("WhatsApp is not installed. Please install WhatsApp to chat.");
				// }
			}, 1000); // Reduced timeout to 1 second
		} else {
			// Desktop - open web.whatsapp.com
			window.open(`https://web.whatsapp.com/send?phone=${phoneNumber}`, '_blank');
		}
	}

</script>

<article class="shadow " style="position:relative">

	<a href={qCard.toViewUrl()} alt="View this QCard">
		<QRCode dataToEncode={selfLink}/>
	</a>

	<hr class="bottom-top-divider">

	<div class="contact-meta ">
		<header>
			<h1>
				{qCard.name}
			</h1>
			{#if qCard.title}
			<h2>{qCard.title}</h2>
			{/if}
		</header>

		<!-- Phone with icon -->
		{#if qCard.phone}
		<div class="detail-divider"></div>
		<div class="detail-container">
			<div class="detail-label">Telephone</div>
			<div class="detail-content">
				<p>{qCard.phone}</p>
				<a href="tel:{qCard.phone}" alt={qCard.phone}>
					<img src="/icons/phone.svg" alt="Phone Icon"/>
				</a>
			</div>
		</div>
		{/if}

		<!-- Email with icon -->
		{#if qCard.email}
		<div class="detail-divider"></div>
		<div class="detail-container">
			<div class="detail-label">Email</div>
			<div class="detail-content">
				<p>{qCard.email}</p>
				<a href="mailto:{qCard.email}" alt="{qCard.email}" target="_blank">
					<img src="/icons/inbox.svg" alt="Email Icon"/>
				</a>
			</div>
		</div>
		{/if}

		<!-- Address with icon -->
		{#if qCard.address}
		<div class="detail-divider"></div>
		<div class="detail-container">
			<div class="detail-label">Address</div>
			<div class="detail-content">
				<p>{qCard.address}</p>
				<a href="https://www.openstreetmap.org/search?query={qCard.address}" target="_blank" alt="{qCard.address}">
					<img src="/icons/map-pin.svg" alt="Location icon"/>
				</a>
			</div>
		</div>
		{/if}

		<!-- Website with icon -->
		{#if qCard.website}
		<div class="detail-divider"></div>
		<div class="detail-container">
			<div class="detail-label">Website</div>
			<div class="detail-content">
				<p>{qCard.website}</p>
				<a href={qCard.website} target="_blank" alt="Website provided by {qCard.name}">
					<img src="/icons/link.svg" alt="External Link Icon"/>
				</a>
			</div>
		</div>
		{/if}

		<!-- WhatsApp Number with icon -->
		{#if qCard.whatsappnumber}
		<div class="detail-divider"></div>
		<div class="detail-container">
			<div class="detail-label">WhatsApp</div>
			<div class="detail-content">
				<p>{qCard.whatsappnumber}</p>
				<!-- svelte-ignore a11y-invalid-attribute -->
				<a href="#" on:click|preventDefault={e => handleWhatsAppClick(qCard.whatsappnumber, e)} alt="Chat on WhatsApp">
					<img src="/icons/whatsapp.svg" alt="WhatsApp icon"/>
				</a>
			</div>
		</div>
		{/if}
	
		<!-- Comment (no icon) -->
		{#if qCard.comment}
		<div class="detail-divider"></div>
		<div class="detail-container">
			<div class="detail-label">Comment</div>
			<div class="detail-content">
				<p>{qCard.comment}</p>
			</div>
		</div>
		{/if}
	</div>

	<hr class="bottom-top-divider">

	<div class="button-container">

		{#if canEdit}
		<button on:click={editQCard} title="Edit the QCard" class="edit">
			<img src="/icons/edit.svg" alt="Edit Icon"/>
		</button>
		{/if}

		{#if navigator.share}
		<button on:click={shareQCard} title="Share this QCard's URL" class="share">
			<img src="/icons/share.svg" alt="Share Icon"/>
		</button>
		{:else}
		<button on:click={copySelfLink} title="Copy the URL of this QCard" class="copy">
			<img src="/icons/copy.svg" alt="Copy Icon"/>
		</button>
		{/if}

		<button on:click={downloadVCard} title="Download the VCard" class="download">
			<img src="/icons/download.svg" alt="Download Icon"/>
		</button>

	</div>
</article>