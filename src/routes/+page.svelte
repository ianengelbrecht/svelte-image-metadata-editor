<script lang="ts">
  import { parseMetadata } from '@uswriting/exiftool';
  import prettyBytes from 'pretty-bytes';
  import ImageCard from '$lib/components/ImageCard.svelte';

type ImageEntry = {
  handle: FileSystemFileHandle;
  file: File;
  metadata: string | null;
  metadataStatus: 'idle' | 'loading' | 'done' | 'error';
  metadataError: string | null;
};

let images: ImageEntry[] = $state([]);

type MetadataDisplay = {
  FileName: string;
  FileSize: number;
  ImageWidth: number;
  ImageHeight: number;
  Make: string;
  Model: string;
  DateTimeOriginal: string;
  GPSLatitude: number | null;
  GPSLongitude: number | null;
  GPSAltitude: number | null;
} | null;
let displayMetadata = $state<MetadataDisplay>(null);

const imageFilePattern = /\.(jpe?g|png|gif|bmp|webp|heic|heif|tif|tiff|avif)$/i;

const handleSelectImagesClick =  async () => {

  const fileOptions = {
    multiple: true,
    types: [
      {
        description: 'Images',
        // MDN link for how to set the 'accept' value
        accept: {
          'image/png': ['.png'],
          'image/jpeg': ['.jpg', '.jpeg'],
          'image/gif': ['.gif'],
          'image/bmp': ['.bmp'],
          'image/webp': ['.webp'],
          'image/heic': ['.heic'],
          'image/heif': ['.heif'],
          'image/tiff': ['.tif', '.tiff'],
          'image/avif': ['.avif'],
        },
      },
    ],
  };


  const fileHandles = await window.showOpenFilePicker(fileOptions);

  for await (const fileHandle of fileHandles) {
    const file = await fileHandle.getFile();
    let metadata: any = null; //just for now
    let metadataStatus: 'idle' | 'loading' | 'done' | 'error' = 'idle';
    let metadataError: string | null = null;
    try {
        const result = await parseMetadata(file, { args: ['-json', '-n'] });
        if (result.success) {
          const json = result.data;
          metadata = JSON.parse(json)[0]; // because exiftool returns an array of results
          metadataStatus = 'done';
        } else {
          metadataError = 'Error parsing metadata: ' + result.error;
          metadataStatus = 'error';
        }
    } catch (err) {
      console.error('Error parsing metadata for file:', file.name, err);
      metadataError = 'Error parsing metadata: ' + String(err);
      metadataStatus = 'error';
    }
    images.push({ handle: fileHandle, file, metadata, metadataStatus, metadataError });
    
  }

};

const setSelectedJSON = (json: MetadataDisplay) => {
  displayMetadata =  json;
};

const addGPXCoordsButtonClick = async () => {
  const fileOptions = {
    multiple: true,
    types: [
      {
        description: 'GPX files',
        // MDN link for how to set the 'accept' value
        accept: {
          'application/gpx+xml': ['.gpx'],
        },
      },
    ],
  };

  let fileHandles: FileSystemFileHandle[] = [];
  try {

    fileHandles = await window.showOpenFilePicker(fileOptions);
  } 
  catch (err) {
    console.error('Error selecting GPX file:', err);
    alert('Error selecting GPX file: ' + String(err));
  }

  for await (const fileHandle of fileHandles) {
    const file = await fileHandle.getFile();

  }   // Here you would parse the GPX file and extract coordinates, then associate them with the images as needed.
  

};

</script>
<div class="p-4 w-screen h-screen grid grid-cols-3 grid-rows-3 gap-4">
  <div class="relative p-4 row-span-3 col-span-2 bg-gray-100 rounded-xl">
    {#if images.length > 0}
      <div class="p-4 inset-0 flex items-center justify-center gap-4 flex-wrap">
        {#each images as image}
          <ImageCard {image} {setSelectedJSON} />
        {/each}
      </div>
    {:else}
      <div class="absolute inset-0 flex flex-col items-center justify-center text-gray-500">
        <div>
          No images uploaded
        </div>
        <button class="p-2 border rounded cursor-pointer" onclick={handleSelectImagesClick}>Select images</button>
      </div>
    {/if}
  </div>
  <div class="relative p-4 bg-gray-100 rounded-xl">
    {#if displayMetadata}
      <!-- <pre>{metadataString}</pre>
      <button aria-label="copy string" class="absolute top-1 right-2 p-2 border rounded cursor-pointer" onclick={() => {
        navigator.clipboard.writeText(metadataString)          .then(() => alert('Metadata copied to clipboard!'))
          .catch(err => alert('Failed to copy metadata: ' + err));
      }}>
        <svg xmlns="http://www.w3.org/2000/svg" width="38" height="8" fill="#000000" viewBox="0 0 256 256"><path d="M216,32H88a8,8,0,0,0-8,8V80H40a8,8,0,0,0-8,8V216a8,8,0,0,0,8,8H168a8,8,0,0,0,8-8V176h40a8,8,0,0,0,8-8V40A8,8,0,0,0,216,32ZM160,208H48V96H160Zm48-48H176V88a8,8,0,0,0-8-8H96V48H208Z"></path></svg>
      </button> -->
    <p>File name: {displayMetadata.FileName}</p>
    <p>File size: {prettyBytes(displayMetadata.FileSize)}</p>
    <p>
      <span>Date: {displayMetadata.DateTimeOriginal?.split(' ')[0].replace(/:/g, "/") || 'N/A'}   </span>
      <span> Time: {displayMetadata.DateTimeOriginal?.split(' ')[1].split(':').slice(0,2).join(':') || 'N/A'}</span>
    </p>
    <p>Coords:  {typeof displayMetadata.GPSLatitude === 'number' ? displayMetadata.GPSLatitude.toFixed(6) : 'N/A'}, 
      {typeof displayMetadata.GPSLongitude === 'number' ? displayMetadata.GPSLongitude.toFixed(6) : 'N/A'}</p>
    <p>Altitude: {typeof displayMetadata.GPSAltitude === 'number' ? Math.round(displayMetadata.GPSAltitude) + 'm' : 'N/A'}</p>
    {:else}
      <div class="flex items-center justify-center h-full text-gray-500">
        No metadata selected
      </div>
    {/if}
  </div>
  <div class="p-4 row-span-2 bg-gray-100 rounded-xl">03</div>
</div>