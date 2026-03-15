<script lang="ts">
  
  let { image, setSelectedJSON } = $props();
  let selected = $state(false)

  const handleImageClick = () => {
    if (image.metadataStatus === 'loading') console.log('metadata still loading'); 
    if (image.metadataStatus === 'error') console.log('metadata error:', image.metadataError);
    if (image.metadataStatus === 'done') {
      try {
        // const json = JSON.parse(image.metadata);
        // const metadata = json[0] // because it's an array
        // const ourData = {
        //   FileName: metadata.FileName,
        //   FileSize: metadata.FileSize,
        //   ImageWidth: metadata.ImageWidth,
        //   ImageHeight: metadata.ImageHeight,
        //   Make: metadata.Make,
        //   Model: metadata.Model,
        //   DateTimeOriginal: metadata.DateTimeOriginal,
        //   GPSLatitude: metadata.GPSLatitude || null,
        //   GPSLongitude: metadata.GPSLongitude || null,
        //   GPSAltitude: metadata.GPSAltitude || null,
        // }
        // setSelectedJSON(ourData);
        setSelectedJSON(image.metadata);
      }
      catch (err) {
        alert('Error parsing metadata JSON: '  + err);
        console.error(image.metadata)
      }
    }
    selected = !selected;
  };

</script>

<button class={
  ["relative w-32 h-32 bg-gray-300 rounded-lg overflow-hidden cursor-pointer", 
  {"opacity-100 border-4 border-blue-500": selected}, 
  {"opacity-50 hover:border-4 hover:border-gray-400": !selected}
  ]} onclick={handleImageClick}>
  {#if image.metadataStatus === 'loading'}
    <span class="absolute top-1 right-1 h-5 w-5 rounded-full bg-yellow-400 flex items-center justify-center text-white" >
      <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" fill="currentColor" viewBox="0 0 256 256"><path d="M128,20a108,108,0,1,0,108,108A108.12,108.12,0,0,0,128,20Zm0,192a84,84,0,1,1,84-84A84.09,84.09,0,0,1,128,212ZM116,60h16v60H116ZM116,132h16v16H116Z"></path></svg>
    </span>
  {:else if image.metadataStatus === 'error'}
    <span class="absolute top-1 right-1 h-5 w-5 rounded-full bg-red-400 flex items-center justify-center text-white" >
      <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" fill="currentColor" viewBox="0 0 256 256"><path d="M128,20a108,108,0,1,0,108,108A108.12,108.12,0,0,0,128,20Zm36.24,135.76a8,8,0,1,1-11.32-11.32L132.69,128l-10.63-10.63a8,8,0,1,1,11.32-11.32L144.69,117l10.63-10.63a8,8,0,1,1,11.32,11.32L156.69,128Z"></path></svg>
    </span>
  {:else if image.metadataStatus === 'done'}
    <span class="absolute top-1 right-1 h-5 w-5 rounded-full bg-green-400 flex items-center justify-center text-white" >
      <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" fill="currentColor" viewBox="0 0 256 256"><path d="M43.18,128a29.78,29.78,0,0,1,8,10.26c4.8,9.9,4.8,22,4.8,33.74,0,24.31,1,36,24,36a8,8,0,0,1,0,16c-17.48,0-29.32-6.14-35.2-18.26-4.8-9.9-4.8-22-4.8-33.74,0-24.31-1-36-24-36a8,8,0,0,1,0-16c23,0,24-11.69,24-36,0-11.72,0-23.84,4.8-33.74C50.68,38.14,62.52,32,80,32a8,8,0,0,1,0,16C57,48,56,59.69,56,84c0,11.72,0,23.84-4.8,33.74A29.78,29.78,0,0,1,43.18,128ZM240,120c-23,0-24-11.69-24-36,0-11.72,0-23.84-4.8-33.74C205.32,38.14,193.48,32,176,32a8,8,0,0,0,0,16c23,0,24,11.69,24,36,0,11.72,0,23.84,4.8,33.74a29.78,29.78,0,0,0,8,10.26,29.78,29.78,0,0,0-8,10.26c-4.8,9.9-4.8,22-4.8,33.74,0,24.31-1,36-24,36a8,8,0,0,0,0,16c17.48,0,29.32-6.14,35.2-18.26,4.8-9.9,4.8-22,4.8-33.74,0-24.31,1-36,24-36a8,8,0,0,0,0-16Z"></path></svg>
    </span>
  {/if}
  <img src={URL.createObjectURL(image.file)} alt={image.file.name} class="object-cover w-full h-full" />
</button>

