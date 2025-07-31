#  *RESTful 201 Web API*
###  *How CreatedAtAction and GetById Work in ASP.NET Core*
<br>
<br>

###  *Note: The 201 Created status or CreatedAtAction return is only used when creating new data only.*

###  *Controller logic*

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using RecordManagementSystem.DTO.Account;
using RecordManagementSystem.Application.Features.Account.Service;
using RecordManagementSystem.Application.Features.Account.DTO;
using AutoMapper;
using System.Threading.Tasks;
using System.ComponentModel.DataAnnotations;
using Microsoft.EntityFrameworkCore.Storage.Internal;

namespace RecordManagementSystem.Controllers.Account
{
    [Route("api/[controller]")]
    [ApiController]
    public class AccountController : ControllerBase
    {
        private readonly AddStudentUserDataServices _services;
        private readonly IMapper _mapper;
        public AccountController(AddStudentUserDataServices services, IMapper mapper)
        {
            _services = services;
            _mapper = mapper;
        }


        [HttpGet("{id}")]
        public ActionResult<AddStudentUserDataDTO> GetId(int id)
        {
            var user = _services.GetIdUsers(id);
            return Ok(user);
        }


        [HttpPost("AddStudentUserData")]
        public async Task<ActionResult> AddStudentUsersData([FromBody]AccountDTO userData) 
        {
            if (ModelState.IsValid)
            {
                var addStudent = _mapper.Map<AddStudentUserDataDTO>(userData);
                var UserId = await _services.AddStudentData(addStudent);

                return CreatedAtAction(nameof(GetId), new { id = UserId.Id }, UserId);
            }
            return BadRequest();
        }


    }
}

```

<br>
<br>

###  *Interface logic*

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using RecordManagementSystem.Application.Features.Account.DTO;
using RecordManagementSystem.Domain.Entities.Account;

namespace RecordManagementSystem.Application.Features.Account.Interface
{
    public interface IAddStudentUserData
    {
        Task<AddStudentUserDataDTO> AddStudent(AddStudentUserDataDTO add);
        Task<AddStudentUserDataDTO> GetUserId(int Id);
    }
}

```
<br>
<br>

###  *Infrastructure repository logic*
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore.Query.Internal;
using RecordManagementSystem.Application.Features.Account.Interface;
using RecordManagementSystem.Infrastructure.Persistence.Data;
using RecordManagementSystem.Application.Features.Account.DTO;
using RecordManagementSystem.Domain.Entities.Account;
using RecordManagementSystem.Application.Common.Models;
using AutoMapper;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Azure;


namespace RecordManagementSystem.Infrastructure.Repository.Features.Account
{
    public class AddStudentUserDataRepository : IAddStudentUserData 
    {
        private readonly ApplicationDbContext _context;
        private readonly UserManager<UserIdentity> _userManager;
        private readonly IMapper _mapper;
        public AddStudentUserDataRepository(IMapper mapper,ApplicationDbContext context, UserManager<UserIdentity> userManager)
        {
            _context = context;
            _userManager = userManager;
            _mapper = mapper;
        }

        public async Task<AddStudentUserDataDTO> GetUserId(int id)
        {
            var UserId = _context.studentUserData.FirstOrDefault(Users => Users.Id == id);

            if(UserId == null) { return null; }

            var dto = new AddStudentUserDataDTO
            {
                Id = UserId.Id,
                FirstName = UserId.FirstName,
                Middlename = UserId.Middlename,
                LastName = UserId.LastName,
                Gender = UserId.Gender,
                YearOfBirth = UserId.YearOfBirth,
                MonthOfBirth = UserId.MonthOfBirth,
                DateOfBirth = UserId.DateOfBirth,
                HomeAddress = UserId.HomeAddress,
                MobileNumber = UserId.MobileNumber,
                Email = UserId.Email,
                Program = UserId.Program,
                YearLevel = UserId.YearLevel,
                StudentID = UserId.StudentID,
                Password = UserId.Password,
            };
        
            return dto;
        }


        public async Task<AddStudentUserDataDTO> AddStudent(AddStudentUserDataDTO addStudentDTO)
        {
       
            var addStudent = _mapper.Map<StudentUserData>(addStudentDTO);
            await _context.AddAsync(addStudent);
            await _context.SaveChangesAsync();

            addStudentDTO.Id = addStudent.Id;
            return addStudentDTO;
        }


    }
}

```


<br>
<br>

###  *Application Services Logic*

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using RecordManagementSystem.Application.Features.Account.DTO;
using RecordManagementSystem.Application.Features.Account.Interface;
using RecordManagementSystem.Domain.Entities.Account;
using RecordManagementSystem.Application.Common.Models;

namespace RecordManagementSystem.Application.Features.Account.Service
{
    public class AddStudentUserDataServices
    {
        private readonly IAddStudentUserData _addStudentUserData;
        public AddStudentUserDataServices(IAddStudentUserData addStudentUserData)
        {
            _addStudentUserData = addStudentUserData;
        }


        public async Task<Result<AddStudentUserDataDTO>> GetIdUsers(int id)
        {
            var UserId = await _addStudentUserData.GetUserId(id);
            if(UserId != null)
            {
                return Result<AddStudentUserDataDTO>.Ok(UserId);
            }
            return Result<AddStudentUserDataDTO>.Fail("Not found!");
        }


        public async Task<AddStudentUserDataDTO> AddStudentData(AddStudentUserDataDTO add)
        {
            return await _addStudentUserData.AddStudent(add);
        }

    }
}

```
