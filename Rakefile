=begin 
Workflow:
  Workflow 1
  Excel => CSV => SQL Scripts => SQLite3 DB
  
  Demos using WebGL?
  Workflow 2
  SQLite3 DB => Swatch Demo of RGB vs CYMK vs HEX vs HSB
  
  Workflow 3
  SQLite3 DB => Demo of Shades
  
  Workflow 4
  SQLite3 DB => Demo of Themes

Proposed Tasks:
  #clobber output dir
  
  gen_csv
  gen_sql
  gen_db => default
  gen_swatches
  gen_shades
  gen_themes
=end

require 'rake/clean'
CLEAN.include('**/*.sql')
CLOBBER.include("output")

task :default => :gen_sql

COLOR_HEADERS = {:ID=>:int,:NAME=>:string,:RED=>:float,:GREEN=>:float,:BLUE=>:float,
  :CYNA=>:float,:YELLOW=>:float,:MAGENTA=>:float,:KEY=>:float,:HUE=>:float,:SATURATION=>:float,
  :BRIGHTNESS=>:float,:HEX=>:string}
  
SHADES_HEADERS = {:ID=>:int,:NAME=>:string,:OPPOSITE=>:string,:SATURATION_START=>:float,
  :SATURATION_END=>:float,:BRIGHTNESS_START=>:float,:BRIGHTNESS_END=>:float,:BLACK_HUE_START=>:float,
  :BLACK_HUE_END=>:float,:BLACK_SATURATION_START=>:float,:BLACK_SATURATION_END=>:float,
  :BLACK_BRIGHTNESS_START=>:float,:BLACK_BRIGHTNESS_END=>:float,:WHITE_HUE_START=>:float,
  :WHITE_HUE_END=>:float,:WHITE_SATURATION_START=>:float,:WHITE_SATURATION_END=>:float,
  :WHITE_BRIGHTNESS_START=>:float,:WHITE_BRIGHTNESS_END=>:float}  
  
CONTEXT_HEADERS = {:ID=>:int,:BLACK=>:string, :BLUE=>:string,	:BROWN=>:string,	:GREEN=>:string,	
  :GREY=>:string, :ORANGE=>:string,	:PINK=>:string, :PURPLE=>:string, :RED=>:string, 
  :WHITE=>:string, :YELLOW=>:string}

ADJECTIVES = {:ID => :int, :THEME => :float, :COLOR_A_NAME => :string,:COLOR_A_WEIGHT => :float,
  :COLOR_A_SHADE_A_NAME => :float, :COLOR_A_SHADE_A_WEIGHT => :float, :COLOR_A_SHADE_B_NAME => :string,
  :COLOR_A_SHADE_B_WEIGHT => :float, :COLOR_A_SHADE_C_NAME => :string, :COLOR_A_SHADE_C_WEIGHT	 => :float,
  :COLOR_B_NAME => :string, :COLOR_B_WEIGHT => :float, :COLOR_B_SHADE_A_NAME => :string,
  :COLOR_B_SHADE_A_WEIGHT => :float, :COLOR_B_SHADE_B_NAME	 => :string,:COLOR_B_SHADE_B_WEIGHT	 => :float,
  :COLOR_B_SHADE_C_NAME => :string, :COLOR_B_SHADE_C_WEIGHT	 => :float, :COLOR_C_NAME	 => :string,
  :COLOR_C_WEIGHT	 => :float, :COLOR_C_SHADE_A_NAME	 => :string, :COLOR_C_SHADE_A_WEIGHT	 => :float,
  :COLOR_C_SHADE_B_NAME	 => :string, :COLOR_C_SHADE_B_WEIGHT	 => :float, :COLOR_C_SHADE_C_NAME	 => :string,
  :COLOR_C_SHADE_C_WEIGHT	 => :float, :COLOR_D_NAME	 => :string, :COLOR_D_WEIGHT	 => :float,
  :COLOR_D_SHADE_A_NAME	 => :string, :COLOR_D_SHADE_A_WEIGHT	 => :float, :COLOR_D_SHADE_B_NAME	 => :string,
  :COLOR_D_SHADE_B_WEIGHT	 => :float, :COLOR_D_SHADE_C_NAME	 => :string, :COLOR_D_SHADE_C_WEIGHT	 => :float,
  :COLOR_E_NAME	 => :string, :COLOR_E_WEIGHT	 => :float, :COLOR_E_SHADE_A_NAME	 => :string,
  :COLOR_E_SHADE_A_WEIGHT	 => :float, :COLOR_E_SHADE_B_NAME	 => :string,:COLOR_E_SHADE_B_WEIGHT	 => :float,
  :COLOR_E_SHADE_C_NAME	 => :string, :COLOR_E_SHADE_C_WEIGHT => :float}

desc "Generate csv files from the master Excel file"
task :gen_sql do
  #create output dir
  require 'rubygems'
  require 'roo'
  
  db_name = 'colors'  
  doc = Excelx.new("./references/colors.xlsx")
  sql = ""
  sql = create_table_from_sheet(doc,"Colors",db_name,'Color', 0, 13,COLOR_HEADERS) +
    create_table_from_sheet(doc,"Shades",db_name,'Shade', 0, 19, SHADES_HEADERS) +
    create_table_from_sheet(doc,"Context",db_name,'Color_Context', 0, 11, CONTEXT_HEADERS) +
    create_table_from_sheet(doc,"Themes Adjectives",db_name,'Themes_Adjectives', 0, 42, ADJECTIVES) +
    create_table_from_sheet(doc,"Themes Basic English",db_name,'Themes_Basic_English', 0, 42, ADJECTIVES) +
    create_table_from_sheet(doc,"Themes Emotion",db_name,'Themes_Emotion', 0, 42, ADJECTIVES) +
    create_table_from_sheet(doc,"Themes Nature",db_name,'Themes_Nature', 0, 42, ADJECTIVES)
        
  mkdir('output')  
  file = File.new("output/create_colors_db.sql",'w')
  file.write(sql)
end

desc "Generate a new SQLite3 database from the sql scripts"
task :gen_db_fast do
  cd 'output'
  exec 'sqlite3 colors.db < create_colors_db.sql'
end

def create_table_from_sheet(doc, sheet_name, db_name, table_name, first_col, last_col,map)
  puts "Generating SQL for #{table_name}"
  doc.default_sheet = sheet_name
  first_row = doc.first_row
  last_row = doc.last_row
  header = doc.row(doc.first_row)[first_col, last_col].map{|m| m.upcase.gsub(' ', '_')}
  
  rows = []
  (first_row+1..last_row).each do |index|
    rows << doc.row(index)[first_col, last_col]
  end
  sql = create_table_with_data(db_name,table_name,header,rows,map,"create the #{table_name} table with data")
  return sql
end

#generate the sql to create a table from an array of rows.
def create_table_with_data(db_name, table_name, header, rows,map,comment)  
  columns_def = process_header(header)
  
  create_table = "create table if not exists #{table_name}(#{columns_def});"
  rows_sql = ""
  rows.each do |row|    
    row_sql = create_insert_sql(db_name,table_name,header,row,map)
    rows_sql = rows_sql + row_sql
  end
  
  sql = %{
--#{comment}    
#{create_table}    

-- Insert data for #{table_name}
#{rows_sql} 
  }
  return sql
end

def process_header(header) 
  str = header * ","
  str.gsub!(' ','_')
  return str
end

def create_insert_sql(db_name,table_name,header,row,map)
  raise StandardError.new("Could not create the insert statement for #{table_name}") if (header.length != row.length)
  
  row_sql = ""
  (0..row.length-1).each do |index|
    column_type = map[header[index].to_sym]
    v = row[index]
    value = case column_type
    when :int then v.to_i
    when :string then (v.nil? || v.empty?)? 'NULL' : "'#{v}'"
    when :float then v.to_f
    else raise StandardError.new("could not determine the column type for column_type: #{column_type} for header: #{header[index].to_sym}")
    end
    row_sql = (index == 0)? value.to_s : row_sql +", "+value.to_s;
  end
  
  columns_def = process_header(header)
  sql = "insert into #{table_name}(#{columns_def}) values(#{row_sql});\n"
  return sql
end