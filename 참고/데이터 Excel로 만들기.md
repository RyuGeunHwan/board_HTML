# ExcelCotroller

```java
package com.dw.board.controller;

import java.net.URLEncoder;
import java.text.SimpleDateFormat;
import java.util.Date;

import javax.servlet.http.HttpServletResponse;

import org.apache.poi.ss.usermodel.Workbook;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

import com.dw.board.sevice.ExcelService;

/**
 * @author dw-004 2022. 6. 16.
 * @comment : excel다운로드 받는 컨트롤러
 */
@Controller
public class ExcelController {

	@Autowired
	private ExcelService excelService;

	// 엑셀, 사진, 한글, PDF, Zip, 영상 파일 등등.. return type이 없음. 에브리바디 모두 void or ResponseEnity
	// 페이지 이름으로 return(X)
	@GetMapping("/excel")
	// try/catch 문법을 여러번 써야할 경우 메소드에 throws Exception를 적어줘서 한번에 잡을 수 있다.
	// HttpServletResponse response를 이용해서 엑셀파일로 데이터를 보냄.
	public void downloadExcelFile(HttpServletResponse response) throws Exception {
		String today = new SimpleDateFormat("yyMMdd").format(new Date());
		String title = "DW아카데미_게시판";

		response.setContentType("ms-vnd/excel");
		response.setHeader("Content-Disposition",
				"attachment;filename=" + URLEncoder.encode(today + "_" + title, "UTF-8") + ".xls");// 엑셀 파일이름 수정
		Workbook workBook = excelService.makeExcelForm();
		workBook.write(response.getOutputStream());
		workBook.close();

		response.getOutputStream().flush();
		response.getOutputStream().close();

	}

}
```

# ExcelService

```java
package com.dw.board.sevice;

import java.util.List;
import java.util.Map;

import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.hssf.util.HSSFColor;
import org.apache.poi.hssf.util.HSSFColor.HSSFColorPredefined;
import org.apache.poi.ss.usermodel.BorderStyle;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.CellStyle;
import org.apache.poi.ss.usermodel.FillPatternType;
import org.apache.poi.ss.usermodel.HorizontalAlignment;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.dw.board.mapper.BoardMapper;

@Service
public class ExcelService {

	@Autowired
	private BoardMapper boardMapper;

	// throws Exception : 이 메소드에서 에러가나면 Exception에서 캐치 해줘~라는 함수
	public Workbook makeExcelForm() throws Exception {

		Workbook workbook = new HSSFWorkbook();// excel 생성
		Sheet sheet = workbook.createSheet("게시판 자료");
		Row row = null; // 엑셀 행
		Cell cell = null; // 엑셀 열
		int rowNumber = 0; // 행 번호

		CellStyle headStyle = makeExcelHeadStyle(workbook);
		CellStyle bodyStyle = makeExcelBodyStyle(workbook);

		row = sheet.createRow(rowNumber++);// 첫번째 행, 엑셀의 행은 1부터 시작함


		cell = row.createCell(0); // 엑셀의 열은 0부터 시작함
		cell.setCellStyle(headStyle);//head style 수정
		cell.setCellValue("게시판번호");// 컬럼명 추가

		cell = row.createCell(1);
		cell.setCellStyle(headStyle);
		cell.setCellValue("작성자");

		cell = row.createCell(2);
		cell.setCellStyle(headStyle);
		cell.setCellValue("제목");

		cell = row.createCell(3);
		cell.setCellStyle(headStyle);
		cell.setCellValue("수정 날짜");

		cell = row.createCell(4);
		cell.setCellStyle(headStyle);
		cell.setCellValue("작성 날짜");

		cell = row.createCell(5);
		cell.setCellStyle(headStyle);
		cell.setCellValue("조회 수");

		// mapper 데이터 호출
		// 기존에 있던 boardMapper를 이용해서 전체조회한 데이터를 사용함.
		List<Map<String, Object>> list = boardMapper.selectBoard();

		for (Map<String, Object> data : list) {
			row = sheet.createRow(rowNumber++);// 행을 계속 추가 해준다. for문 조건식이 만족할 때 까지.


			cell = row.createCell(0);// 게시판 번호
			cell.setCellStyle(bodyStyle);//body style 수정
			cell.setCellValue(data.get("boardId").toString());//컬럼명에 맞는 데이터 추가

			cell = row.createCell(1);// 작성자
			cell.setCellStyle(bodyStyle);
			cell.setCellValue(data.get("studentsId").toString());

			cell = row.createCell(2);// 제목
			cell.setCellStyle(bodyStyle);
			cell.setCellValue(data.get("title").toString());

			cell = row.createCell(3);// 수정 날짜
			cell.setCellStyle(bodyStyle);
			cell.setCellValue(data.get("updateAt").toString());

			cell = row.createCell(4);// 작성 날짜
			cell.setCellStyle(bodyStyle);
			cell.setCellValue(data.get("createAt").toString());

			cell = row.createCell(5);// 조회 수
			cell.setCellStyle(bodyStyle);
			cell.setCellValue(data.get("cnt").toString());

		}

		return workbook;
	}

	//엑셀 head style 수정
		public CellStyle makeExcelHeadStyle(Workbook workbook) {
			CellStyle cellStyle = null;
			cellStyle = workbook.createCellStyle();
			//가는 경계선 생성
			cellStyle.setBorderTop(BorderStyle.THIN);
			cellStyle.setBorderLeft(BorderStyle.THIN);
			cellStyle.setBorderRight(BorderStyle.THIN);
			cellStyle.setBorderBottom(BorderStyle.THIN);
			//배경색 생성
			cellStyle.setFillForegroundColor(HSSFColor.HSSFColorPredefined.YELLOW.getIndex());
			cellStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
			//가운데 정렬
			cellStyle.setAlignment(HorizontalAlignment.CENTER);
			return cellStyle;
		}


		//엑셀 body style 수정
		public CellStyle makeExcelBodyStyle(Workbook workbook) {
			CellStyle cellStyle = null;
			cellStyle = workbook.createCellStyle();
			//가는 경계선 생성
			cellStyle.setBorderTop(BorderStyle.THIN);
	        cellStyle.setBorderBottom(BorderStyle.THIN);
	        cellStyle.setBorderLeft(BorderStyle.THIN);
	        cellStyle.setBorderRight(BorderStyle.THIN);
			return cellStyle;
		}

}

```
