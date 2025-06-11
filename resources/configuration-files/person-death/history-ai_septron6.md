
copied orm.yaml from ai_septron7 newly generated
#edited to just person & death tables

datafaker configure-generators
#p propose
#s 2 set to option2
#for keys did next 

datafaker make-stats

datafaker create-tables
datafaker create-generators --stats-file src-stats.yaml
## got to here next step failed because foreign key tables not present (e.g. concept)
## but I can still copy the config files into the repo
datafaker create-data
datafaker create-data --num-passes 2

