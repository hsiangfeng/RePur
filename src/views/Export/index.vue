<template src="./template.html" />

<script setup>
import axios from 'axios';
import { ref, onMounted } from 'vue';
import ExcelJS from 'exceljs';
import { saveAs } from 'file-saver';
import ApiErrorMsg from '../../components/ApiErrorMsg.vue';

const channelOptions = ref([]);

// msg
const isApiErrorMsg = ref(false);

const showApiErrorMsg = (error) => {
  isApiErrorMsg.value = true;
  setTimeout(() => {
    isApiErrorMsg.value = false;
  }, 3000);

  console.log(error);
};

const apiGetChannelOptions = () => axios.get('/api/GET/channels');
const getChannelOptions = async () => {
  try {
    const { data } = await apiGetChannelOptions();
    channelOptions.value = data;
  } catch (error) {
    showApiErrorMsg(error);
  }
};

const exportExcel = async (dataList) => {
  const workbook = new ExcelJS.Workbook();
  workbook.addWorksheet('Product', {
    views: [
      {
        activeCell: 'A1',
        showGridLines: false,
        state: 'frozen',
        ySplit: 1,
      },
    ],
  });

  const worksheet = workbook.getWorksheet(1);
  // worksheet.properties.defaultRowHeight = 50; // row height
  // worksheet.getRow(2).height = 50;
  worksheet.columns = [
    { header: 'Id', key: 'id', width: 10 },
    { header: 'Name', key: 'name', width: 32 },
    { header: 'Image', key: 'img', width: 10 },
  ];

  // Alignment
  const columnIdx = [1, 2, 3];
  const firstRows = worksheet.getRow(1);
  columnIdx.forEach((col) => {
    firstRows.getCell(col).alignment = {
      vertical: 'middle',
      horizontal: 'center',
    };
  });

  for (let i = 0; i < dataList.length; i += 1) {
    worksheet.addRow({ id: dataList[i].id, name: dataList[i].name });

    const rows = worksheet.getRow(i + 2);
    rows.height = 50;
    columnIdx.forEach((col) => {
      rows.getCell(col).alignment = {
        vertical: 'middle',
        horizontal: 'center',
      };
    });

    const base64url = `${dataList[i].img}`;
    // console.log('base64url', base64url);
    if (base64url !== '') {
      const imageId = workbook.addImage({
        base64: base64url,
        extension: 'jpg',
      });
      worksheet.addImage(imageId, {
        tl: { col: 2, row: i + 1 },
        ext: { width: 50, height: 50 },
        hyperlinks: {
          hyperlink: 'https://www.google.com.tw/',
          tooltip: 'https://www.google.com.tw/',
        },
      });
    }

    if (i === dataList.length - 1) {
      const fileType = 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet';
      const downloadDate = new Date().toISOString().substring(0, 10).replaceAll('-', '');
      const filename = `${downloadDate}下載_RePur_商品紀錄.xlsx`;
      workbook.xlsx.writeBuffer().then((data) => {
        const blob = new Blob([data], { type: fileType });
        saveAs(blob, filename);
      });
    }
  }
};

const apiGetProductList = () => axios.get('/api/GET/products');
// eslint-disable-next-line no-unused-vars
const getProductList = async () => {
  try {
    const { data } = await apiGetProductList();
    const dataList = [];
    data.map((item) => {
      const obj = { id: item.id, name: item.name, img: item.pic };
      dataList.push(obj);

      const parseText = (value) => {
        if (value === '1') return '全聯';
        if (value === '2') return '家樂福';
        return '其他';
      };

      return {
        ...item,
        sales_channel_text: parseText(item.sales_channel),
      };
    });

    exportExcel(dataList);
  } catch (error) {
    showApiErrorMsg(error);
  }
};

// life-cycle
onMounted(() => {
  getChannelOptions();
});
</script>

<script>
export default {
  name: 'ExportView',
};
</script>

<style scoped>
a {
  color: #42b983;
}
</style>
