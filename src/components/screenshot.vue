<script>
import pica from 'pica';
import SmartCrop from 'smartcrop';
import { useTheme } from 'vuetify';

export default {
  methods: {
    async processScreenshot(searchText) {
      if (!searchText) {
        console.error("No text found in search box");
        return null; // Return null if there's no text
      }

      const charWidth = 20;
      const canvasHeight = 50;
      const canvasWidth = (charWidth * searchText.replace(/\s/g, '').length) + 18; // 9px margin on each side
      const margin = 9;

      const tempCanvas = document.createElement('canvas');
      const context = tempCanvas.getContext('2d');
      tempCanvas.width = canvasWidth;
      tempCanvas.height = canvasHeight + 18; // 9px margin on top and bottom

      // Set background color based on theme
      const theme = useTheme();
      theme.global.name.value = localStorage.getItem("theme") || "dark";
      const isDarkMode = theme.global.name.value === "dark";
      context.fillStyle = isDarkMode ? 'black' : 'white';
      context.fillRect(0, 0, tempCanvas.width, tempCanvas.height);

      // Draw your content on tempCanvas here, with margin
      context.fillStyle = isDarkMode ? 'white' : 'black';
      context.font = '20px indus_scriptregular'; // Set font size and family
      context.fillText(searchText, margin, margin + 40); // Adjust text position based on margin

      // Use SmartCrop to detect the white pixels and crop the rest
      const result = await SmartCrop.crop(tempCanvas, { width: canvasWidth, height: canvasHeight });
      const crop = result.topCrop;

      const croppedCanvas = document.createElement('canvas');
      croppedCanvas.width = crop.width;
      croppedCanvas.height = crop.height;
      const croppedContext = croppedCanvas.getContext('2d');
      croppedContext.drawImage(tempCanvas, crop.x, crop.y, crop.width, crop.height, 0, 0, crop.width, crop.height);

      // Use Pica to resize the cropped canvas if needed
      const picaInstance = pica();
      const resizedCanvas = await picaInstance.resize(croppedCanvas, { width: canvasWidth, height: canvasHeight });

      return resizedCanvas;
    }
  }
};

export const downloadScreenshot = async () => {
  try {
    const searchBox = document.querySelector('#search');
    if (!searchBox) {
      console.error("Search box not found");
      return;
    }

    const searchText = searchBox.value;
    const resizedCanvas = await processScreenshot(searchText);

    // Do something with resizedCanvas, e.g., download it
    const link = document.createElement('a');
    link.href = resizedCanvas.toDataURL('image/png');
    link.download = 'screenshot.png';
    link.click();
  } catch (error) {
    console.error("Error processing screenshot:", error);
  }
};
</script>