# Database and Software Engineering and Design Enhancement   
   
[artifact](https://github.com/AshleyJohnson90/courses-original){:target="_blank" rel="noopener"}   
[enhancement](https://github.com/AshleyJohnson90/CoursesPage){:target="_blank" rel="noopener"}   
   
<video controls autoplay muted width="1000" height="750">
  <source src="../assets/App-Demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
   
**Briefly describe the artifact. What is it? When was it created?**   
   
The original artifact is a program written in C++ that, from the terminal, allows the user to load a list of classes read from a file. From there they can make a selection to see a list of all the classes or search for a class. If the class is found in the loaded data, it will show the class name and prerequisites if applicable. This artifact was created in my first year of the program in the class CS300 Data Structures and Algorithms.   
   
**Justify the inclusion of the artifact in your ePortfolio. Why did you select this item? What specific components showcase your skills and abilities in software development? How was the artifact improved?**     
   
I chose this artifact because it shows how I can take a basic, abstract idea of a program and re-engineer it into an application that can be widely used by SNHU students. I improved the artifact by turning it into a full stack web application, porting from C++ to C# and using .NET framework and a PostgreSQL database as the backend and React with Javascript as the frontend.   

The user can see a table containing all the classes with their prerequisites and can sort by subject.   
```
        <>
            <div>
                <FormControl variant="outlined" fullWidth margin="normal">
                    <InputLabel>Filter by Subject</InputLabel>
                    <Select
                        value={chosenSubject}
                        onChange={(e) => setChosenSubject(e.target.value)}
                        label="Subject">
                        <MenuItem value=""><em>All Subjects</em></MenuItem>
                        {[...new Set(tableData.map(course => course.subject))].map(subject => (
                            <MenuItem key={subject} value={subject}>{subject}</MenuItem>
                        ))}
                    </Select>
                </FormControl>
            </div>
            <div>
            <Paper elevation={7}>
                    <Box id="table-box" sx={{ height: '60vh', overflow: 'auto', borderRadius: '10px' }}>

                    <TableContainer>
                    <Table id="table">
                        <TableHead>
                            <TableRow>
                                <TableCell style={{ width: '3%' }} />
                                <TableCell
                                    onClick={() => handleSort('subject')}
                                    style={{ width: '15%', fontSize: '1rem' }}
                                >
                                    Subject
                                    <ExpandMore
                                        sx={{
                                            fontSize: 20,
                                            background: 'none',
                                            ml: 1,
                                            transition: '0.3s',
                                            transform: orderBy === 'subject' ? (sortDirection === 'asc' ? 'rotate(0deg)' : 'rotate(180deg)') : 'rotate(0deg)',
                                        }}
                                    />
                                </TableCell>
                                <TableCell style={{ width: '10%', fontSize: '1rem' }}>Course ID</TableCell>
                                <TableCell style={{ width: '30%', fontSize: '1rem' }}>Course Name</TableCell>
                                <TableCell style={{ width: '33%', fontSize: '1rem' }}>Prerequisite(s)</TableCell>
                            </TableRow>
                        </TableHead>
                        <TableBody>
                            {filteredData.map((row) => (
                                <CollapsibleRow key={row.id} row={row} />
                            ))}
                        </TableBody>
                    </Table>
                </TableContainer>
                </Box>
            </Paper>
            </div>
        </>
```
   
The table is loaded with course data entered into the CoursesPage Postgres database. The table shows each course subject, course ID, course name, and prerequisites. Each course also has a drop-down menu where the user can read the description of the course.   
   
```
        <React.Fragment>
            <TableRow>
                <TableCell>
                    <IconButton aria-label="expand row" size="medium" onClick={() => setOpen(!open)}>
                        {open ? <KeyboardArrowUp /> : <KeyboardArrowDown />}
                    </IconButton>
                </TableCell>
                <TableCell>{row.subject}</TableCell>
                <TableCell sx={{ fontWeight: 'bold'}}>{row.id}</TableCell>
                <TableCell sx={{ fontWeight: 'bold'}}>{row.name}</TableCell>
                <TableCell>{row.prerequisite}</TableCell>
            </TableRow>
            <TableRow>
                <TableCell style={{ paddingBottom: 0, paddingTop: 0 }} colSpan={5}>
                    <Collapse in={open} timeout="auto" unmountOnExit>
                        <Box sx={{ margin: 1 }}>
                            <Typography variant="h5" gutterBottom component="div">Description</Typography>
                            <p>{row.description}</p>
                        </Box>
                    </Collapse>
                </TableCell>
            </TableRow>
        </React.Fragment>
```

The original artifact used a binary search tree to store the course data. This would not be sufficient storage for a webpage. The data needs to be persistent and easy to query so a database would be the best approach for data storage.   

I had to create a connection between .NET and the Postgres database so they could communicate with each other.
```
private readonly IConfiguration _configuration;

public CoursesDatabaseContext(IConfiguration configuration)
{
	_configuration = configuration;
}

public CoursesDatabaseContext(DbContextOptions<CoursesDatabaseContext> options, IConfiguration configuration)
	: base(options)
{
	_configuration = configuration;
}

public virtual DbSet<Course> Courses { get; set; }

protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
	if (!optionsBuilder.IsConfigured)
	{
		var connectionString = _configuration.GetConnectionString("PostgresDatabase");
		optionsBuilder.UseNpgsql(connectionString);
	}
}
```

The database could then be constructed using the Entity framework   

```
	protected override void OnModelCreating(ModelBuilder modelBuilder)
	{
		modelBuilder.Entity<Course>(entity =>
		{
			entity.HasKey(e => e.Id).HasName("courses_pkey");

			entity.ToTable("courses");

			entity.Property(e => e.Id).HasColumnName("id");
			entity.Property(e => e.Description).HasColumnName("description");
			entity.Property(e => e.Name).HasColumnName("name");
			entity.Property(e => e.Prerequisite).HasColumnName("prerequisite");
			entity.Property(e => e.Subject).HasColumnName("subject");
		});

		OnModelCreatingPartial(modelBuilder);
	}

	partial void OnModelCreatingPartial(ModelBuilder modelBuilder);
}
```

To query the database and give the webpage the information, I had to create a REST API to communicate with the database. It uses the HTTP GET request to get all the course information that the webpage needs to display. It also allows the table to be sorted by subject but using the HTTP GET method and a subject name to show the courses for the chosen subject.   

```
namespace CoursesPage.Server.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class TableController : ControllerBase
    {
        private readonly CoursesDatabaseContext _context;
        public TableController(CoursesDatabaseContext context)
        {
            _context = context;
        }

        [HttpGet]

        public async Task<IActionResult> GetAll()
        {
            var allCourses = await _context.Courses.ToListAsync();

            return Ok(allCourses);
        }

        [HttpGet("{subject}")]

        public async Task<IActionResult> GetBySubject([FromRoute] string subject)
        {
            var course = await _context.Courses.Where(c => c.Subject == subject).ToListAsync();
            return Ok(course);
        }
    }
    
}
```
   
**Did you meet the course outcomes you planned to meet with this enhancement in Module One? Do you have any updates to your outcome-coverage plans?**   
   
I did meet the course outcomes I planned to meet with this enhancement. My enhancements were to meet the outcomes:   
1. Demonstrate an ability to use well-founded and innovative techniques, skills, and tools in computing practices for the purpose of implementing computer solutions that deliver value and accomplish industry-specific goals.
2. Design, develop, and deliver professional-quality oral, written, and visual communications that are coherent, technically sound, and appropriately adapted to specific audiences and contexts.

I have effectively used innovative technologies by using concepts that were brand new to me to create this enhancement. I have no previous experience using React, C#, .NET, or PostgreSQL. These are modern, relevant technologies used by companies large and small to create webpages. The course listing page implements a professional quality visual computer solution that delivers value to students who are searching for classes to take. In just a few clicks they can find out important information about the university's classes.   
   
**Reflect on the process of enhancing and modifying the artifact. What did you learn as you were creating it and improving it? What challenges did you face?**   
   
I’ve learned quite a lot while enhancing the artifact. I’ve gotten to practice using the SPA structure to create a full stack application to maintain separation of concerns. I’ve learned about using UI libraries like React and then about open-source component projects built upon React, like Radix and Material UI. I’ve learned how important it is to plan out your application and read documentation of the libraries I used. It’s been a lot of information to take in at one time but I’m learning a lot of valuable lessons to take into a future career. I’ve faced challenges with styling the table where I store the classes. I didn’t realize there were defaults that I couldn’t just change with CSS, I had to create themes that override each default I want to change. This has taught me to inspect computed elements to find the name of the element so I can change it and to also see what is styling it that I do or do not want.   

The database connects with the backend using a REST API. I learned how to create the API using C# and which HTTP request I need to get my data into my application. I did run into some challenges when creating the API and connecting the database. In the program, there are different ports for HTTPS and HTTP. When I first was setting things up, it was connecting using HTTP when I needed it to use HTTPS but I did not realize this. The terminal was saying there were two different ports open which would then make my database not connect to my application. Because this was my first time ever using .NET and Postgres, I didn’t know what to look for to know the database wasn’t connected properly. It took quite a bit of debugging time to figure out why I couldn’t load my data.
