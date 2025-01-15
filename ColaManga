// ==UserScript==
// @name         ColaManga Image Downloader
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  Tải ảnh từ ColaManga
// @author       You
// @match        https://www.colamanga.com/manga-*/*/*.html
// @grant        none
// @require      https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js
// @require      https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js
// ==/UserScript==

(function () {
    'use strict';

    // Tạo nút "Bắt đầu tải ảnh"
    const button = document.createElement('button');
    button.innerText = 'Bắt đầu tải ảnh';
    button.style.position = 'fixed';
    button.style.top = '50%';
    button.style.left = '50%';
    button.style.transform = 'translate(-50%, -50%)';
    button.style.zIndex = 9999;
    button.style.padding = '20px';
    button.style.fontSize = '20px';
    button.style.backgroundColor = '#28a745';
    button.style.color = 'white';
    button.style.border = 'none';
    button.style.borderRadius = '10px';
    button.style.cursor = 'pointer';
    document.body.appendChild(button);

    let isDownloading = false;

    // Hàm tải ảnh qua canvas và lưu vào file zip
    async function downloadImagesToZip() {
        const containers = [...document.querySelectorAll('.mh_mangalist .mh_comicpic')];

        // Sắp xếp các container theo thứ tự thuộc tính "p"
        containers.sort((a, b) => {
            const pA = parseInt(a.getAttribute('p'), 10);
            const pB = parseInt(b.getAttribute('p'), 10);
            return pA - pB;
        });

        const totalImages = containers.length;
        let downloadedImages = 0;
        const zip = new JSZip();

        for (let i = 0; i < containers.length; i++) {
            const container = containers[i];
            const img = container.querySelector('img');
            const isLoading = container.querySelector('.mh_loading');

            if (img && img.complete && !isLoading) {
                // Tải ảnh qua canvas
                const canvas = document.createElement('canvas');
                const context = canvas.getContext('2d');

                canvas.width = img.naturalWidth;
                canvas.height = img.naturalHeight;
                context.drawImage(img, 0, 0);

                const dataUrl = canvas.toDataURL('image/jpeg');
                const blob = await (await fetch(dataUrl)).blob();

                zip.file(`image_${i + 1}.jpg`, blob);
                downloadedImages++;
            }

            // Cập nhật tiến trình trên nút
            button.innerText = `Đã tải ảnh thứ ${downloadedImages}`;

            // Đợi một chút trước khi tải ảnh tiếp theo để tránh lỗi hoặc bỏ sót
            await new Promise(resolve => setTimeout(resolve, 500));
        }

        // Lưu file zip
        button.innerText = 'Đang nén file...';
        zip.generateAsync({ type: 'blob' }).then((content) => {
            saveAs(content, 'images.zip');
            button.innerText = "Đã tải xong!";
            button.disabled = false;
            //button.innerText = 'Bắt đầu tải ảnh';
            isDownloading = false;
        });
    }

    // Hàm tự động cuộn trang để load tất cả ảnh
    async function autoScrollToBottom() {
        return new Promise((resolve) => {
            button.innerText = 'Đang cuộn ảnh...';

            const interval = setInterval(() => {
                const scrollHeight = document.body.scrollHeight;
                const scrollTop = window.scrollY;
                const clientHeight = window.innerHeight;

                if (scrollTop + clientHeight >= scrollHeight) {
                    clearInterval(interval);
                    resolve();
                } else {
                    window.scrollBy(0, 200);
                }
            }, 100);
        });
    }

    // Hàm chính khi nhấn nút
    async function start() {
        if (isDownloading) return;
        isDownloading = true;
        button.disabled = true;
        button.innerText = 'Đang cuộn ảnh...';

        console.log("Đang cuộn trang để load tất cả ảnh...");
        await autoScrollToBottom(); // Cuộn xuống cuối trang

        console.log("Bắt đầu tải ảnh và lưu vào file zip...");
        await downloadImagesToZip(); // Tải ảnh và nén thành file zip
    }

    // Thêm sự kiện click cho nút
    button.addEventListener('click', start);
})();
